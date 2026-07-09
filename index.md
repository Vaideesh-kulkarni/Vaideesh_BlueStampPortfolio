# Ball Tracking Robot with OpenCV: Computer Vision

The Ball Tracking Robot with OpenCV uses a Raspberry Pi 4 computer, a 5MP camera, and Python to create a robot that avoids obstacles, moves independently, and makes decisions about where to navigate. This project involves building circuits to make connections, integrating hardware and software, and programming.

| **Engineer** | **School** | **Area of Interest** | **Grade** |
|:---|:---|:---|:---|
| Vaideesh K | Cupertino High School | Electrical Engineering | Incoming Senior |

![Headshot](images/headshot.jpg)
---

# Final Milestone

<iframe width="560" height="315" src="https://www.youtube.com/embed/F7M7imOVGug" title="Final Milestone" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

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

# Code

## Basic Code for the Motor Driver

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

## Basic Code for the Ultrasonic Sensors

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

---

# Other Resources / Examples

One of the best parts about GitHub is that you can view how other people set up their own work.

- [Claude Ai help Project](https://claude.ai/share/d7c5e0e6-0105-45f7-9baa-1b5a52f87b24)
- [Portfolio Help — Derin's BSE Portfolio](https://deringur.github.io/BSE_Derin_Portfolio/)
- [Raspberry Pi 4 Model B Pin Layout](https://www.raspberrypi.com/documentation/computers/raspberry-pi.html#gpio)
