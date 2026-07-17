from flask import Flask, Response, render_template_string
from picamera2 import Picamera2
import cv2
import numpy as np
import RPi.GPIO as GPIO
import time
import threading

# ================= TUNING =================
# motors
MIN_SPEED    = 55
CRUISE_SPEED = 100
TURN_SPEED   = 90
AVOID_SPEED  = 85
SLOW_FROM    = 60
ARRIVE_AT    = 15
OBSTACLE_AT  = 20
SIDE_EVERY   = 8

# detection
MIN_AREA   = 400
MIN_RADIUS = 20
MAX_RADIUS = 100
MIN_FILL   = 0.55
LOST_HOLD  = 10

# servo camera pan
SERVO_DIR = -1        # flip to 1 if camera turns away from the ball
SERVO_GAIN = 0.03
SERVO_MAX_STEP = 3
SERVO_DEADZONE = 55
SERVO_MIN = 20
SERVO_MAX = 160

# ================= MOTORS =================
GPIO.setmode(GPIO.BCM)
GPIO.setwarnings(False)
A1A = 6; A1B = 5; B1A = 22; B2A = 23
for p in [A1A, A1B, B1A, B2A]:
    GPIO.setup(p, GPIO.OUT)
pA1A = GPIO.PWM(A1A, 1000); pA1B = GPIO.PWM(A1B, 1000)
pB1A = GPIO.PWM(B1A, 1000); pB2A = GPIO.PWM(B2A, 1000)
for p in [pA1A, pA1B, pB1A, pB2A]:
    p.start(0)

INSIDE = 0.4
def forward(s):
    pA1A.ChangeDutyCycle(0); pA1B.ChangeDutyCycle(s)
    pB1A.ChangeDutyCycle(0); pB2A.ChangeDutyCycle(s)
def left(s):
    pA1A.ChangeDutyCycle(0); pA1B.ChangeDutyCycle(int(s * INSIDE))
    pB1A.ChangeDutyCycle(0); pB2A.ChangeDutyCycle(s)
def right(s):
    pA1A.ChangeDutyCycle(0); pA1B.ChangeDutyCycle(s)
    pB1A.ChangeDutyCycle(0); pB2A.ChangeDutyCycle(int(s * INSIDE))
def stop():
    for p in [pA1A, pA1B, pB1A, pB2A]:
        p.ChangeDutyCycle(0)

# ================= SERVO =================
SERVO = 18
GPIO.setup(SERVO, GPIO.OUT)
servo_pwm = GPIO.PWM(SERVO, 50)
servo_pwm.start(0)
cam_angle = 90.0
def drive_servo(a):
    a = max(SERVO_MIN, min(SERVO_MAX, a))
    duty = 2.5 + (a / 180.0) * 10.0
    servo_pwm.ChangeDutyCycle(duty)
    time.sleep(0.02)
    servo_pwm.ChangeDutyCycle(0)
    return a
cam_angle = drive_servo(cam_angle)

# ================= SENSORS =================
SENSORS = {"LEFT": (19, 26), "CENTER": (16, 20), "RIGHT": (11, 12)}
for trig, echo in SENSORS.values():
    GPIO.setup(trig, GPIO.OUT)
    GPIO.setup(echo, GPIO.IN)
    GPIO.output(trig, False)
def measure(trig, echo):
    start = time.time(); stop_t = time.time()
    GPIO.output(trig, True); time.sleep(0.00001); GPIO.output(trig, False)
    t = time.time() + 0.006
    while GPIO.input(echo) == 0 and time.time() < t: start = time.time()
    t = time.time() + 0.006
    while GPIO.input(echo) == 1 and time.time() < t: stop_t = time.time()
    d = round((stop_t - start) * 34300 / 2, 1)
    return d if 0 < d < 400 else 400

# ================= CAMERA =================
picam2 = Picamera2()
picam2.configure(picam2.create_preview_configuration(
    main={"format": "RGB888", "size": (480, 360)}))
picam2.start()
time.sleep(2)

W = 480
CENTER = W // 2
LEFT_EDGE  = W // 3
RIGHT_EDGE = W * 2 // 3
kernel = np.ones((5, 5), np.uint8)

running = False
latest = None
lock = threading.Lock()
dL = dR = 400
frame_count = 0
sx = None
lost = 0

def speed_for(d):
    if d >= SLOW_FROM: return CRUISE_SPEED
    if d <= ARRIVE_AT: return MIN_SPEED
    frac = (d - ARRIVE_AT) / float(SLOW_FROM - ARRIVE_AT)
    return int(MIN_SPEED + frac * (CRUISE_SPEED - MIN_SPEED))

def brain():
    global latest, dL, dR, frame_count, sx, lost, cam_angle
    while True:
        frame_count += 1
        frame = picam2.capture_array()
        frame = cv2.flip(frame, -1)

        dC = measure(*SENSORS["CENTER"])
        if frame_count % SIDE_EVERY == 0:
            dL = measure(*SENSORS["LEFT"])
            dR = measure(*SENSORS["RIGHT"])

        # ---- DETECTION ----
        blurred = cv2.GaussianBlur(frame, (5, 5), 0)
        hsv = cv2.cvtColor(blurred, cv2.COLOR_BGR2HSV)
        lo1 = np.array([0, 150, 90]);   hi1 = np.array([10, 255, 255])
        lo2 = np.array([170, 150, 90]); hi2 = np.array([180, 255, 255])
        mask = cv2.inRange(hsv, lo1, hi1) + cv2.inRange(hsv, lo2, hi2)
        mask = cv2.morphologyEx(mask, cv2.MORPH_OPEN,  kernel)
        mask = cv2.morphologyEx(mask, cv2.MORPH_CLOSE, kernel, iterations=2)
        mask = cv2.copyMakeBorder(mask, 1, 1, 1, 1, cv2.BORDER_CONSTANT, value=0)

        contours, _ = cv2.findContours(mask, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)
        best = None; best_score = 0
        for c in contours:
            area = cv2.contourArea(c)
            if area < MIN_AREA:
                continue
            (mx, my), mr = cv2.minEnclosingCircle(c)
            if mr < MIN_RADIUS or mr > MAX_RADIUS:
                continue
            fill = area / (np.pi * mr * mr) if mr > 0 else 0
            if fill < MIN_FILL:
                continue
            score = fill * area
            if score > best_score:
                best_score = score
                best = (int(mx), int(my), int(mr))

        # smooth + hold
        if best is not None:
            mx = best[0]
            sx = mx if sx is None else 0.6 * sx + 0.4 * mx
            lost = 0
        else:
            lost += 1
            if lost > LOST_HOLD:
                sx = None

        ball = sx is not None
        pos = "NONE"
        if ball:
            cx = int(sx)
            if cx < LEFT_EDGE:    pos = "LEFT"
            elif cx > RIGHT_EDGE: pos = "RIGHT"
            else:                 pos = "CENTER"
            if best is not None:
                bx, by, br = best
                cv2.circle(frame, (bx, by), br, (0, 255, 0), 3)

            # ---- SERVO: pan camera to keep ball centered ----
            err = cx - CENTER
            if abs(err) > SERVO_DEADZONE:
                stepc = SERVO_DIR * SERVO_GAIN * err
                stepc = max(-SERVO_MAX_STEP, min(SERVO_MAX_STEP, stepc))
                cam_angle = cam_angle + stepc
        cam_angle = drive_servo(cam_angle)

        # ---- DRIVE ----
        if not running:
            stop(); action = "STOPPED"
        elif dL < OBSTACLE_AT and dL < dR:
            right(AVOID_SPEED); action = "avoid"
        elif dR < OBSTACLE_AT:
            left(AVOID_SPEED);  action = "avoid"
        elif ball:
            if 0 < dC <= ARRIVE_AT:
                stop(); action = "ARRIVED"
            elif pos == "LEFT":
                left(TURN_SPEED);  action = "left"
            elif pos == "RIGHT":
                right(TURN_SPEED); action = "right"
            else:
                s = speed_for(dC)
                forward(s); action = f"fwd {s}%"
        else:
            stop(); action = "no ball"

        cv2.line(frame, (LEFT_EDGE, 0),  (LEFT_EDGE, 360),  (80, 80, 80), 1)
        cv2.line(frame, (RIGHT_EDGE, 0), (RIGHT_EDGE, 360), (80, 80, 80), 1)
        cv2.putText(frame, f"{pos} | {action}", (10, 30),
                    cv2.FONT_HERSHEY_SIMPLEX, 0.7, (0, 255, 0), 2)
        cv2.putText(frame, f"C{dC}  cam={int(cam_angle)}", (10, 60),
                    cv2.FONT_HERSHEY_SIMPLEX, 0.6, (0, 255, 0), 2)

        ok, jpg = cv2.imencode('.jpg', frame, [int(cv2.IMWRITE_JPEG_QUALITY), 60])
        if ok:
            with lock:
                latest = jpg.tobytes()

threading.Thread(target=brain, daemon=True).start()

app = Flask(__name__)
PAGE = """
<html><head><title>Ball Tracking Robot</title>
<style>
 body{background:#111;color:#eee;font-family:sans-serif;text-align:center}
 img{border:3px solid #444;border-radius:8px;margin-top:15px;width:90%;max-width:600px}
 button{font-size:20px;padding:14px 34px;margin:10px;border:none;border-radius:6px;cursor:pointer}
 .go{background:#2a7;color:#fff} .no{background:#a33;color:#fff}
</style></head>
<body>
 <h1>Ball Tracking Robot</h1>
 <button class="go" onclick="fetch('/start')">START</button>
 <button class="no" onclick="fetch('/stop')">STOP</button>
 <br><img src="/video">
</body></html>
"""

def gen():
    while True:
        with lock:
            d = latest
        if d is None:
            time.sleep(0.03); continue
        yield (b'--frame\r\nContent-Type: image/jpeg\r\n\r\n' + d + b'\r\n')
        time.sleep(0.03)

@app.route('/')
def index(): return render_template_string(PAGE)

@app.route('/start')
def go():
    global running; running = True; return "started"

@app.route('/stop')
def halt():
    global running; running = False; stop(); return "stopped"

@app.route('/video')
def video():
    return Response(gen(), mimetype='multipart/x-mixed-replace; boundary=frame')

try:
    app.run(host='0.0.0.0', port=5000, threaded=True)
finally:
    stop(); servo_pwm.stop(); GPIO.cleanup()
