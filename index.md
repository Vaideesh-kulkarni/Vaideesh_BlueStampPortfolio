# Ball Tracking Robot with OpenCV: Computer Vision

The Ball Tracking Robot with OpenCV uses a Raspberry Pi 4 computer, a 5MP camera, and Python to create a robot that avoids obstacles, moves independently, and makes decisions about where to navigate. This project involves building circuits to make connections, integrating hardware and software, and programming.

| **Engineer** | **School** | **Area of Interest** | **Grade** |
|:---|:---|:---|:---|
| Vaideesh K | Cupertino High School | Electrical Engineering | Incoming Senior |

![Headshot](Vaideesh%20K.jpg)

---

# Final Milestone

**Summary**

My third and final milestone is the last portion of my project. In this part I installed a 5MP Raspberry Pi Camera and used OpenCV to run the vision code. The finished robot combines the three ultrasonic sensors, the L9110 motor driver, and the two motors so that it can detect and follow a red ball while using the three ultrasonic sensors to measure the distance to objects on the left, center, and right, and to detect and avoid obstacles in real time. The camera sees the ball and decides whether it is on the left, center, or right, and the robot turns or drives forward to follow it, stopping when it gets close. To make sure the robot only follows the ball and not any other red object, I added a circularity check that measures how round each red object is, so only round objects like the ball are tracked.

**Challenges**

A major challenge I faced was getting the camera to be detected by the Raspberry Pi 4 Model B. I ran `rpicam-hello` to turn on the camera and check for a live preview, but it did not work. My next step was to completely power down the Pi by unplugging the USB-C cable and reseating the camera ribbon cable at both the camera module and the connector on the Raspberry Pi. I rebooted and ran it again, but it still did not work, so I tried three different cameras of the same model with different ribbon cables. I ran `rpicam-hello --list-cameras` to see if anything would show up, but no cameras were available. I then ran an update to see if that would fix the issue, but nothing changed. As a last resort I created a camera test — I made a file, wrote the camera code, saved it, and ran it, and it finally worked.

Another challenge was making the robot track only the red ball and not every red object in the room. At first the code just picked the largest red blob, so it would follow red shirts or anything else red. I fixed this by adding a circularity calculation that compares each object's area to its perimeter to measure how round it is. I also had to clean up the mask with morphological operations, because glare on the ball punched holes in the detected shape and ruined the roundness math. I also had to fix the camera orientation, since the image was coming in upside down.

**Camera Test Code**

This is the camera test I used to confirm the camera was working with OpenCV. It grabs frames from the Pi Camera using picamera2 and displays them in a live window. Getting this to run was the fix that finally got my camera working after it wouldn't show a preview.

```python
from picamera2 import Picamera2
import cv2

picam2 = Picamera2()
config = picam2.create_preview_configuration(main={"format": "RGB888", "size": (320, 240)})
picam2.configure(config)
picam2.start()

print("Camera running - press q in the window to quit")
while True:
    frame = picam2.capture_array()
    cv2.imshow("Camera", frame)
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break

cv2.destroyAllWindows()
picam2.stop()
```

**Ball Detection Code**

After the camera worked, I wrote this code to detect the red ball. It converts each frame to HSV, filters for red, cleans up the mask so glare doesn't break the shape, and then calculates the circularity of each red object. Only round objects count as the ball, so other red objects get ignored. The code draws a green box when it finds the ball and prints whether the ball is on the left, center, or right.

```python
from picamera2 import Picamera2
import cv2
import numpy as np

picam2 = Picamera2()
config = picam2.create_preview_configuration(main={"format": "RGB888", "size": (320, 240)})
picam2.configure(config)
picam2.start()

kernel = np.ones((5, 5), np.uint8)

# Roundness threshold - raise it if it tracks non-ball red objects,
# lower it if it misses the ball
MIN_CIRCULARITY = 0.4

print("Detecting red ball - press q to quit")
while True:
    frame = picam2.capture_array()
    frame = cv2.flip(frame, -1)

    blurred = cv2.GaussianBlur(frame, (5, 5), 0)
    hsv = cv2.cvtColor(blurred, cv2.COLOR_BGR2HSV)

    lower1 = np.array([0, 150, 80])
    upper1 = np.array([10, 255, 255])
    lower2 = np.array([170, 150, 80])
    upper2 = np.array([180, 255, 255])
    mask = cv2.inRange(hsv, lower1, upper1) + cv2.inRange(hsv, lower2, upper2)

    # Clean the mask - glare punches holes in the ball and wrecks the
    # perimeter math, so closing fills them back in
    mask = cv2.morphologyEx(mask, cv2.MORPH_OPEN, kernel)
    mask = cv2.morphologyEx(mask, cv2.MORPH_CLOSE, kernel, iterations=3)
    mask = cv2.dilate(mask, kernel, iterations=1)

    contours, _ = cv2.findContours(mask, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)

    best = None
    best_area = 0
    fallback = None
    fallback_area = 0

    for c in contours:
        area = cv2.contourArea(c)
        if area < 300:
            continue

        perimeter = cv2.arcLength(c, True)
        if perimeter == 0:
            continue
        circularity = 4 * np.pi * area / (perimeter * perimeter)

        # Track the biggest round one
        if circularity >= MIN_CIRCULARITY and area > best_area:
            best = (c, circularity)
            best_area = area

        # Also track the biggest blob overall, in case nothing is round
        if area > fallback_area:
            fallback = (c, circularity)
            fallback_area = area

    chosen = best if best is not None else fallback

    if chosen is not None:
        c, circ = chosen
        area = cv2.contourArea(c)
        x, y, w, h = cv2.boundingRect(c)
        cx = x + w // 2

        # Green box = passed roundness (it's the ball)
        # Red box = failed roundness (probably not the ball)
        color = (0, 255, 0) if circ >= MIN_CIRCULARITY else (0, 0, 255)
        cv2.rectangle(frame, (x, y), (x+w, y+h), color, 2)

        if cx < 107:
            pos = "LEFT"
        elif cx > 213:
            pos = "RIGHT"
        else:
            pos = "CENTER"

        label = f"{pos}  circ={circ:.2f}"
        cv2.putText(frame, label, (10, 30), cv2.FONT_HERSHEY_SIMPLEX, 0.7, color, 2)
        print(f"Ball: {pos}  area: {int(area)}  circularity: {circ:.2f}")

    cv2.imshow("Ball Tracking", frame)

    if cv2.waitKey(1) & 0xFF == ord('q'):
        break

cv2.destroyAllWindows()
picam2.stop()
```

**Final Code — Full Ball Tracking Robot**

This is the complete program that combines the camera, the motors, and the ultrasonic sensors. The camera detects the red ball, checks that it is round so other red objects are ignored, and decides if the ball is on the left, center, or right. Based on that, the robot turns or drives forward to follow the ball, and the center ultrasonic sensor stops the robot when it gets close so it does not crash into the ball.

```python
from picamera2 import Picamera2
import cv2
import numpy as np
import RPi.GPIO as GPIO
import time

# ---------- MOTOR SETUP ----------
GPIO.setmode(GPIO.BCM)
GPIO.setwarnings(False)

A1A = 6; A1B = 5; B1A = 22; B2A = 23
for p in [A1A, A1B, B1A, B2A]:
    GPIO.setup(p, GPIO.OUT)

def forward():
    GPIO.output(A1A, GPIO.LOW); GPIO.output(A1B, GPIO.HIGH)
    GPIO.output(B1A, GPIO.LOW); GPIO.output(B2A, GPIO.HIGH)
def backward():
    GPIO.output(A1A, GPIO.HIGH); GPIO.output(A1B, GPIO.LOW)
    GPIO.output(B1A, GPIO.HIGH); GPIO.output(B2A, GPIO.LOW)
def left():
    GPIO.output(A1A, GPIO.LOW); GPIO.output(A1B, GPIO.LOW)
    GPIO.output(B1A, GPIO.LOW); GPIO.output(B2A, GPIO.HIGH)
def right():
    GPIO.output(A1A, GPIO.LOW); GPIO.output(A1B, GPIO.HIGH)
    GPIO.output(B1A, GPIO.LOW); GPIO.output(B2A, GPIO.LOW)
def stop():
    for p in [A1A, A1B, B1A, B2A]: GPIO.output(p, GPIO.LOW)

# ---------- SENSOR SETUP ----------
sensors = [("LEFT", 19, 26), ("CENTER", 16, 20), ("RIGHT", 11, 12)]
for name, trig, echo in sensors:
    GPIO.setup(trig, GPIO.OUT)
    GPIO.setup(echo, GPIO.IN)
    GPIO.output(trig, False)

def measure(trig, echo):
    start = time.time()
    stop_t = time.time()
    GPIO.output(trig, True)
    time.sleep(0.00001)
    GPIO.output(trig, False)
    t = time.time() + 0.05
    while GPIO.input(echo) == 0 and time.time() < t:
        start = time.time()
    t = time.time() + 0.05
    while GPIO.input(echo) == 1 and time.time() < t:
        stop_t = time.time()
    return round((stop_t - start) * 34300 / 2, 1)

# ---------- CAMERA SETUP ----------
picam2 = Picamera2()
config = picam2.create_preview_configuration(main={"format": "RGB888", "size": (320, 240)})
picam2.configure(config)
picam2.start()
time.sleep(2)

kernel = np.ones((5, 5), np.uint8)

# Roundness threshold - raise it if it tracks non-ball red objects,
# lower it if it misses the ball
MIN_CIRCULARITY = 0.4

print("Ball tracking robot running - press q to quit")

try:
    while True:
        frame = picam2.capture_array()
        frame = cv2.flip(frame, -1)

        blurred = cv2.GaussianBlur(frame, (5, 5), 0)
        hsv = cv2.cvtColor(blurred, cv2.COLOR_BGR2HSV)

        lower1 = np.array([0, 150, 80])
        upper1 = np.array([10, 255, 255])
        lower2 = np.array([170, 150, 80])
        upper2 = np.array([180, 255, 255])
        mask = cv2.inRange(hsv, lower1, upper1) + cv2.inRange(hsv, lower2, upper2)

        # Clean the mask - fills glare holes so the roundness math works
        mask = cv2.morphologyEx(mask, cv2.MORPH_OPEN, kernel)
        mask = cv2.morphologyEx(mask, cv2.MORPH_CLOSE, kernel, iterations=3)
        mask = cv2.dilate(mask, kernel, iterations=1)

        center_dist = measure(16, 20)

        contours, _ = cv2.findContours(mask, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)

        # Find the biggest ROUND red blob (the ball)
        best = None
        best_area = 0
        fallback = None
        fallback_area = 0

        for c in contours:
            area = cv2.contourArea(c)
            if area < 300:
                continue
            perimeter = cv2.arcLength(c, True)
            if perimeter == 0:
                continue
            circularity = 4 * np.pi * area / (perimeter * perimeter)

            if circularity >= MIN_CIRCULARITY and area > best_area:
                best = (c, circularity)
                best_area = area

            if area > fallback_area:
                fallback = (c, circularity)
                fallback_area = area

        chosen = best if best is not None else fallback

        ball_found = False
        pos = "NONE"
        circ = 0.0

        if chosen is not None:
            c, circ = chosen
            # Only treat it as the ball if it passed the roundness test
            if circ >= MIN_CIRCULARITY:
                ball_found = True

            area = cv2.contourArea(c)
            x, y, w, h = cv2.boundingRect(c)
            cx = x + w // 2

            # Green box = it's the ball. Red box = red thing, not round enough.
            color = (0, 255, 0) if ball_found else (0, 0, 255)
            cv2.rectangle(frame, (x, y), (x+w, y+h), color, 2)

            if cx < 107:
                pos = "LEFT"
            elif cx > 213:
                pos = "RIGHT"
            else:
                pos = "CENTER"

        # ---------- DECISION LOGIC ----------
        if ball_found:
            if 0 < center_dist < 15:
                stop()
                action = "ARRIVED - stopped"
            elif pos == "LEFT":
                left()
                action = "turning left"
            elif pos == "RIGHT":
                right()
                action = "turning right"
            else:
                forward()
                action = "driving forward"
        else:
            stop()
            action = "searching (no ball)"

        cv2.putText(frame, f"{pos} | {action}", (10, 30),
                    cv2.FONT_HERSHEY_SIMPLEX, 0.6, (0, 255, 0), 2)
        cv2.putText(frame, f"dist: {center_dist} cm  circ: {circ:.2f}", (10, 60),
                    cv2.FONT_HERSHEY_SIMPLEX, 0.6, (0, 255, 0), 2)
        cv2.imshow("Ball Tracking Robot", frame)

        print(f"Ball: {pos} | {action} | dist: {center_dist} cm | circ: {circ:.2f}")

        if cv2.waitKey(1) & 0xFF == ord('q'):
            break

except KeyboardInterrupt:
    pass

stop()
cv2.destroyAllWindows()
picam2.stop()
GPIO.cleanup()
print("Stopped and cleaned up")
```

---

# Second Milestone

<iframe width="560" height="315" src="https://www.youtube.com/embed/lQya6fe888A" title="Vaideesh K. Milestone 2" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

**Summary**

My second milestone was the biggest portion of my project. I had to get the motors spinning, which in turn makes the wheels spin, and I had to get all three ultrasonic sensors working. I wrote code for both — the motor code drives the robot forward, backward, left, and right, and the sensor code uses the Pi to send a signal to the Trigger pin, releasing a burst of ultrasonic sound at approximately 40,000 Hz. The sound travels out, hits an object, and bounces back, and the time it takes to return is used to calculate the distance.

**Challenges**

The biggest challenge I faced was getting the motors running. I tried four different L298N motor driver boards because the first one arrived defective — it was missing all of its screw terminals and had pads that were already soldered shut, which made it very difficult to connect the battery and motor wires reliably and safely.

The problem that took about two weeks to solve was supplying power to the motors. The board would not power on through the +12V input while I was using the L298N. I tried 6V, 7.5V, and 9V battery packs, and nothing worked. I had to systematically test each part — the battery holder, the batteries, the motors, the Pi, and the wiring — to isolate exactly where the problem was.

Another issue was unreliable connections, which was the hardest part because everything *looked* correct, but I eventually realized some wires weren't making solid contact with the Raspberry Pi or the breadboard. I was also missing a few connections because I was progressing too quickly.

After all of these problems, there was one simple fix that would have saved a lot of time: switching to the L9110 motor driver. The L9110 uses a single power input instead of separate logic and motor inputs, which makes everything much simpler — and once I switched, the motors finally spun.

**Motor Driver Code**

This code uses basic WASD controls to move the robot, which is useful for testing the most basic mechanics of the motors. It also confirms that the wiring to the motor driver and the Raspberry Pi is correct. The HIGH/LOW combinations create different patterns, which cause the changes in direction.

```python
import RPi.GPIO as GPIO
GPIO.setmode(GPIO.BCM)

A1A = 6; A1B = 5; B1A = 22; B2A = 23
for p in [A1A, A1B, B1A, B2A]:
    GPIO.setup(p, GPIO.OUT)

def forward():
    GPIO.output(A1A, GPIO.LOW); GPIO.output(A1B, GPIO.HIGH)
    GPIO.output(B1A, GPIO.LOW); GPIO.output(B2A, GPIO.HIGH)
def backward():
    GPIO.output(A1A, GPIO.HIGH); GPIO.output(A1B, GPIO.LOW)
    GPIO.output(B1A, GPIO.HIGH); GPIO.output(B2A, GPIO.LOW)
def left():
    GPIO.output(A1A, GPIO.LOW); GPIO.output(A1B, GPIO.LOW)
    GPIO.output(B1A, GPIO.LOW); GPIO.output(B2A, GPIO.HIGH)
def right():
    GPIO.output(A1A, GPIO.LOW); GPIO.output(A1B, GPIO.HIGH)
    GPIO.output(B1A, GPIO.LOW); GPIO.output(B2A, GPIO.LOW)
def stop():
    for p in [A1A, A1B, B1A, B2A]: GPIO.output(p, GPIO.LOW)

print("w=forward s=back a=left d=right x=stop q=quit")
try:
    while True:
        u = input("move: ")
        if u == 'w': forward()
        elif u == 's': backward()
        elif u == 'a': left()
        elif u == 'd': right()
        elif u == 'x': stop()
        elif u == 'q': break
except KeyboardInterrupt:
    pass
GPIO.cleanup()
```

**Ultrasonic Sensor Code**

This code tests whether the ultrasonic sensors work. It measures the distance to an object by firing a pulse from each sensor, timing how long the echo takes to return, and using the speed of sound to calculate the distance. It reads all three sensors — left, center, and right — and prints their distances.

```python
import RPi.GPIO as GPIO
import time
GPIO.setmode(GPIO.BCM)
GPIO.setwarnings(False)

# All 3 sensors: (name, TRIG, ECHO)
sensors = [
    ("LEFT",   19, 26),
    ("CENTER", 16, 20),
    ("RIGHT",  11, 12),
]

for name, trig, echo in sensors:
    GPIO.setup(trig, GPIO.OUT)
    GPIO.setup(echo, GPIO.IN)
    GPIO.output(trig, False)

time.sleep(2)

def measure(trig, echo):
    start = time.time()
    stop = time.time()
    GPIO.output(trig, True)
    time.sleep(0.00001)
    GPIO.output(trig, False)
    timeout = time.time() + 0.05
    while GPIO.input(echo) == 0 and time.time() < timeout:
        start = time.time()
    timeout = time.time() + 0.05
    while GPIO.input(echo) == 1 and time.time() < timeout:
        stop = time.time()
    return round((stop - start) * 34300 / 2, 1)

try:
    while True:
        for name, trig, echo in sensors:
            d = measure(trig, echo)
            print(name, ":", d, "cm")
        print("-----")
        time.sleep(0.5)
except KeyboardInterrupt:
    GPIO.cleanup()
```

**What's Next**

Connect the camera to the Raspberry Pi and add code so it can track the ball. I will write the OpenCV code so the robot can detect the red ball, then combine the camera, sensors, and motors so it can track the ball and avoid obstacles in its path.

---

# First Milestone

<iframe width="560" height="315" src="https://www.youtube.com/embed/zEN702sMDmo" title="Vaideesh K. Milestone 1" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

**Summary**

The goal of the first milestone was to build the foundation of the ball-tracking robot: assembling the robot chassis, getting the Raspberry Pi running with the necessary software installed, and completing all of the electronic connections (wiring the two motors to the L9110 motor driver, the three ultrasonic sensors, and the power). The Pi drives the motors, and the camera is added so the robot can track the red ball and follow it.

**Components Used**

- **Raspberry Pi 4 Model B** — the brain behind everything. It runs the code and controls the robot, from the sensors to the motors.
- **PiCamera (OV5647)** — the camera that detects the ball (used in later milestones).
- **L9110 Motor Driver** — an H-bridge board that lets the Pi control the motors. I switched to this after using the L298N.
- **2× Yellow TT Motors** — spin the wheels to move the robot.
- **2× Wheels + Front Caster** — the wheels drive the robot, and the caster helps balance it and mount the front sensors.
- **3× HC-SR04 Ultrasonic Sensors** — measure distance for obstacle detection.
- **Resistors (1kΩ and 2kΩ)** — build the voltage dividers that protect the Pi's 3.3V pins from the sensors' 5V signals.
- **Breadboard** — the main hub for connecting the voltage dividers, power, and sensor wiring.
- **Jumper Wires** — connect all the components together.
- **4×AA Battery Pack (6V)** — powers the motors.
- **USB-C Powerbank** — powers the Pi.
- **Clear Acrylic 2WD Chassis** — the frame that holds everything together.

**Challenges**

The challenges I faced were finding diagrams to help me make the electrical connections and making all of the connections myself. There were nearly 40 connections to make, and the wires kept getting tangled, the resistors kept getting unplugged, and everything was disorganized. I had to reseat and redo cables multiple times, but in the end I got them organized. I also ran into a problem where some cables were dead, so I used a multimeter to find and replace them.

**What's Next**

Make the motors work when connected to the motor driver, and make the ultrasonic sensors detect the distance of an object placed in front of them.

---

# Schematics

**Ball Tracking Robot Diagram**

<img src="Ball%20Tracking%20Robot%20With%20Open%20CV%20Schematics.jpg" width="600">

**Robot Build Photos**

<img src="IMG_6778.jpeg" width="300"> <img src="IMG_6781.jpeg" width="300"> <img src="IMG_6782.jpeg" width="300">

---

# Bill of Materials

| **Part** | **Note** | **Price** | **Link** |
|:---|:---|:---|:---|
| Raspberry Pi 4 Model B | Small computer used for controlling the robot and writing code | $79.97 | [Link](https://www.amazon.com/Raspberry-Model-2019-Quad-Bluetooth/dp/B07TC2BK1X) |
| Raspberry Pi Camera Module | Camera used for live video and seeing the ball | $14.99 | [Link](https://www.amazon.com/Arducam-Autofocus-Raspberry-Motorized-Software/dp/B07SN8GYGD) |
| L298N Driver Board | A basic motor driver used to drive the wheels forward and backward | $8.99 | [Link](https://www.amazon.com/Qunqi-2Packs-Controller-Stepper-Arduino/dp/B01M29YK5U) |
| Motors and Board Kit | Basic hardware pieces that help with the assembly of the robot | $13.59 | [Link](https://www.amazon.com/Smart-Chassis-Motors-Encoder-Battery/dp/B01LXY7CM3) |
| Powerbank | Supplies power to the Raspberry Pi 4 | $21.98 | [Link](https://www.amazon.com/Anker-Ultra-Compact-High-Speed-VoltageBoost-Technology/dp/B07QXV6N1B) |
| HC-SR04 Sensors (5 pcs) | Used for distance calculations of objects and obstacles | $8.99 | [Link](https://www.amazon.com/Organizer-Ultrasonic-Distance-MEGA2560-ElecRight/dp/B07RGB4W8V) |
| HDMI to Micro HDMI Cable | Connects the Raspberry Pi 4 to a laptop for the OBS video stream | $8.99 | [Link](https://www.amazon.com/UGREEN-Adapter-Ethernet-Compatible-Raspberry/dp/B06WWQ7KLV) |
| Video Capture Card | Necessary to display the Pi's output on laptops | $16.99 | [Link](https://www.amazon.com/Capture-Streaming-Broadcasting-Conference-Teaching/dp/B09FLN63B3) |
| SD Card Reader | Necessary to flash the microSD card and install an OS | $4.99 | [Link](https://www.amazon.com/Reader-Adapter-Camera-Memory-Wansurs/dp/B0B9QZ4W4Y) |
| Wired Mouse and Keyboard | Needed to operate the Raspberry Pi 4 | $25.99 | [Link](https://www.amazon.com/Wireless-Keyboard-Trueque-Cordless-Computer/dp/B09J4RQFK7) |
| Basic Connections Components Kit | Includes male-to-male, female-to-female, and male-to-female jumper wires, resistors, and LEDs | $11.47 | [Link](https://www.amazon.com/Smraza-Breadboard-Resistors-Mega2560-Raspberry/dp/B01HRR7EBG) |
| Soldering Kit | Used for the motor connections | $13.60 | [Link](https://www.amazon.com/Soldering-Interchangeable-Adjustable-Temperature-Enthusiast/dp/B087767KNW) |

---

# Starter Project: Retro Arcade

<iframe width="560" height="315" src="https://www.youtube.com/embed/BF2v-AP0EPY" title="Retro Arcade Starter Project" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

**Summary**

My starter project was the Retro Arcade Console. It works by receiving input from the buttons on the front display, which is processed through the integrated circuits and displayed on an LCD screen. It also includes a buzzer for sound effects and runs on batteries. After I soldered all of the electronic components onto the circuit board, I could play games like Tetris and Snake, with the system keeping track of the player's score. This project taught me how to solder electronics to circuit boards, identify electronic components, and troubleshoot electrical connections. Building the Retro Arcade Console helped prepare me for my main project, the Ball Tracking Robot.

**Components Used**

- Microcontroller
- Small LCD Screen
- Buzzer
- Capacitors
- 4×AA Battery Holder / AA Batteries
- PCB (Circuit Board)
- Solder Kit
- Header Pins
- Acrylic Case
- Variety of Colored Buttons

**Challenges Faced**

There were several challenges, some harder than others. My first challenge was soldering the board — every time I soldered, the solder kept bridging to other holes, which could cause a short circuit and damage the board. Another problem was getting the red and black battery wires to sit neatly in two tiny holes and holding them in place so I could solder them properly. I had to unsolder many parts multiple times because the console simply would not turn on. But after all of these hardships, I managed to fix every one of them and get the console working properly.
