# Ball Tracking Robot with OpenCV: Computer Vision
The Ball Tracking Robot with OpenCv uses a Rasberry pi 4 computer, a 5mp camera, and python to create a robot that avoids obstacles, moves independently, and can make decisions on where to navigate. This project involves using circuits to make connections, hardware and software integrations, as well as programming.

| Vaideesh K | Cupertino Highschool | Electrical Engineering | Incoming Senior |
![Headstone Image](logo.svg)
# Final Milestone
<iframe width="560" height="315" src="https://www.youtube.com/embed/F7M7imOVGug" frameborder="0" allowfullscreen></iframe>

# Second Milestone
<iframe width="828" height="474" src="https://www.youtube.com/embed/lQya6fe888A" title="Vaideesh K. Milestone 2" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

Summary: 

My second milestone was the biggest portion of my project. I had to get the motors spinning which in turn would make the wheels spin. I also had to get the 3 ultrasonic sensors working as well. I also made code for both to spin the motors either forward, backward, left, and right. I also made code for the ultrasonic sensors as well which uses the pi to send a signal to the trig to release a burst of ultrasonic sounds approxiametley 40,000 hz. The sound travels out then hit the object bounces back and  you get the distance. 

Challenges: 

Some challenges that I faced was getting the motors running that was the biggest problem. I tried 4 different L298N motor driver board because the first one arrived as defective and it was missing all the screw terminals as well as was pre soldered which made it very very hard to even connect the battery and the motor wires reliably and safe. 

The biggest problem that tooks a 2 weeks or so was supplying power to the motor. The board would not power on through the 12v input while using the L298N board I tried using the 6v, 7.5v, and then the 9v batteyr packs and nothing was working. I had to systematically test each part which included the battery holder the batteries, the motors, the pi, and then the wiring to see where the exact problem was. 

Another issue was unreliable connections which was the hardest part because I thought that everything looked correct but in the end I relaized that some wires werent making correct connections to the rasberry pi board or to the breadboard. I was also missing some connections as I was progressing way to fast. 

After all these problems there was one simple fix that I could've done a while back which wouldve saved a bunch of time and that was by changing the motor driver to the I9110 motordriver. The L9110 uses a single power input instead of multiplee seerate inputs and logic which makes everything much more simpler. 

What's next:

Connect the camera to the Rasberry pi and add code so that the camera can track the ball. I will also write the OpenCV code so that the robot can detect the red ball and then combine the camera, sensors, and motors all together so it can track the ball and avoid any obstacles in the way.




# First Milestone

<iframe width="1177" height="662" src="https://www.youtube.com/embed/zEN702sMDmo" title="Vaideesh K. Milestone 1" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

Summary

The goal of the first milestone was to build the foundation of the ball tracking robot project which includes assembling the robot chasiss, get the Rabpberry Pi running installing the nesscary softwares as well as rcompelting all the elctronic connections which includes (wiring the double motors to the I9110 motor driver, the 3 ultrasonic sensors, and power). The pi drives the motors, the camera is added onto the pi to be able to track the red ball and make the robot follow it as well. 

Components Used:

Raspberry Pi 4 Model B: This is the brain behind everything. It runs the code and controls everything about the robot from the sensors to the motors.

PiCamera(OV5647): This is the camera that helps in detecting the ball which is used in the later milestons

L9110 Motor Driver: This H bridge board is used to let the pi controol the motors I switched this after using the L298N motor driver

2x yellow tt motors: These spin the wheels in order to make the robot move

2x wheels and front caster: These wheels drive the robot and that caster helps mount the ultrasonic sensors on the frotn

3x HC-SRO4 Ultranoic Sensors: These are used to measure the distance for obstacle detection

Resiotrs (1k and 2k): These are used to build the voltagae dividors and they help protect the pi 3.3v from the 5v sensor signals. 

Breadboard: This is the main hub for connecting all the voltage dividers, the power, and the sensor wiring.

Jumper Wires: these help connect all the components together.

4 AA BAttery Pack(6v): This helps to power the motors on and spin the motors.

USB-C Powerbank: Helps to power on the pi and make the robot move.

Clear Acrylic 2wd Chasis: This is the frame that holds everything together in place. 

Challenges: 

Challenges that I faced was finding diagrams that would help me make the elctrical connections in the project as well as making the connections myself. There were many connections I had to make almost 40 and wires were getting tangled, the resisotrs were getting unplugged and so much unorganization. I had to reseat and redo cables multiple times but in the end I got organiized cables. There was also the problem in that some of the cables were dead so I had to use the multimeter to help me in that regard. 

What's next: 

Make the motors work when connected to the motordrvier as well as make the ultrasonic sensors detect the distance of an object when placed in front of it. 

# Schematics

# Basic code for motor driver
```import RPi.GPIO as GPIO
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
        if u=='w': forward()
        elif u=='s': backward()
        elif u=='a': left()
        elif u=='d': right()
        elif u=='x': stop()
        elif u=='q': break
except KeyboardInterrupt:
    pass
GPIO.cleanup()

```


# Basic code for ultrasonic sensors
```import RPi.GPIO as GPIO
import time
GPIO.setmode(GPIO.BCM)

TRIG = 11   # GPIO 11 = pin 23
ECHO = 12   # GPIO 12 = pin 32 (through divider)

GPIO.setup(TRIG, GPIO.OUT)
GPIO.setup(ECHO, GPIO.IN)
GPIO.output(TRIG, False)
time.sleep(2)

try:
    while True:
        GPIO.output(TRIG, True)
        time.sleep(0.00001)
        GPIO.output(TRIG, False)

        start = time.time()
        stop = time.time()
        while GPIO.input(ECHO) == 0:
            start = time.time()
        while GPIO.input(ECHO) == 1:
            stop = time.time()

        distance = (stop - start) * 34300 / 2
        print("Distance:", round(distance, 2), "cm")
        time.sleep(0.5)
except KeyboardInterrupt:
    GPIO.cleanup()

```

# Bill of Materials
| **Part** | **Note** | **Price** | **Link** |
|:--:|:--:|:--:|:--:|
| Raspberri Pi 4 Model B	 | small computer that is used for controling the robot as well as typing code | $Price | Link |
| Raspberry Pi Camera Module | Camera is used for live video and seeing the ball | $Price | Link |
| L298N Driver Board | A basic motor driver that is used to drive the wheels forward and backward. | $Price | Link |
|Motors and Board kit | Basic hardware pieces that help with the assembly of the robot | $Price | Link |
| Powerbank| To supply power to the rasberry pi 4 | $Price | Link |
| HC-SR04 sensors (5 pcs)	 | Used for the difstance calulations of objects and obstacles that are unwanted. | $Price | Link |
| HDMI to micro HDMI cable	 | Connecting the Rasberri pi 4 to the laptop to show on OBS video stream | $Price | Link |
| Video Capture card | Nessecary to display onto laptops | $Price | Link |
| SD card reader | Nessecary to flash microSD and install an os onto it | $Price | Link |
| Wired Mouse and Keyboard | Seperate mouse and keyboard needed to operate the Raberri Pi 4 | $Price | Link |
| Basic connections components kit | Includes nesscary components such as the male to male and female to female as well as the male to female jumper wires, resistors, and LED'S. | $Price | Link |
| Soldering Kit |Soldering Kit is used for the motor connections. | $Price | Link |

# Starter Project: Retro Arcade
<iframe width="560" height="315" src="https://www.youtube.com/embed/BF2v-AP0EPY?si=9xm23H1ms8oinXFz" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
My starter project was the retro arcade console. How it works is basically by receiving input by all the buttons on the front display which get processed through the integrated circuits which displays the game on a LED display. It also includes a buzzer for sound effects as well as runs on batteries. After I soldered all the electronic components onto the circuit board, I could play games such as Tetris and Snake with the system helping keep track of the player's score. This project taught me how to solder electronics to circuit boards, identify electronic components, and troubleshoot electrical connections. Making the Retro Arcade Console helped prepare me a lot for my main project, the Ball Tracking Robot.

# Other Resources/Examples
One of the best parts about Github is that you can view how other people set up their own work.
- [Example 1](https://trashytuber.github.io/YimingJiaBlueStamp/)
- [Example 2](https://sviatil0.github.io/Sviatoslav_BSE/)
- [Example 3](https://arneshkumar.github.io/arneshbluestamp/)
To watch the BSE tutorial on how to create a portfolio, click here.
