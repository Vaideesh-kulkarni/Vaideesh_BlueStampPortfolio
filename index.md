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

This code used basic wasd controls to move the robot which is very useful for testing the most basic mechanics of the motors as well. It also helps to confirm that the wiring is correct to the motor driver as well as the Rasberry pi 4 model B. The HIGH/LOW combinations causes different patterns which causes the changes in direction. 


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

What this code does is test if the ultrasonic sensors work. The code measures the distance to an object using 


```cat > sensortest.py << 'EOF'
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
EOF
```

# Bill of Materials
| **Part** | **Note** | **Price** | **Link** |
|:--:|:--:|:--:|:--:|
| Raspberri Pi 4 Model B	 | small computer that is used for controling the robot as well as typing code | $79.97 | [Link](https://www.amazon.com/Raspberry-Model-2019-Quad-Bluetooth/dp/B07TC2BK1X?source=ps-sl-shoppingads-lpcontext&ref_=fplfs&smid=A2QE71HEBJRNZE&th=1) |
| Raspberry Pi Camera Module | Camera is used for live video and seeing the ball | $14.99 | [Link](https://www.amazon.com/Arducam-Autofocus-Raspberry-Motorized-Software/dp/B07SN8GYGD/ref=sr_1_5?crid=3236VFT39VAPQ&keywords=picamera&qid=1689698732&s=electronics&sprefix=picamer%2Celectronics%2C138&sr=1-5) |
| L298N Driver Board | A basic motor driver that is used to drive the wheels forward and backward. | $8.99 | [Link](https://www.amazon.com/Qunqi-2Packs-Controller-Stepper-Arduino/dp/B01M29YK5U/ref=sr_1_1_sspa?crid=3DE9ZH0NI3KJX&keywords=l298n&qid=1689698859&s=electronics&sprefix=l298n%2Celectronics%2C164&sr=1-1-spons&sp_csd=d2lkZ2V0TmFtZT1zcF9hdGY&psc=1) |
|Motors and Board kit | Basic hardware pieces that help with the assembly of the robot | $13.59 | [Link](https://www.amazon.com/Smart-Chassis-Motors-Encoder-Battery/dp/B01LXY7CM3/ref=sr_1_4?crid=27ACD61NPNLO4&keywords=robot+car+kit&qid=1689698962&s=electronics&sprefix=robot+car+kit%2Celectronics%2C169&sr=1-4) |
| Powerbank| To supply power to the rasberry pi 4 | $21.98 | [Link](https://www.amazon.com/Anker-Ultra-Compact-High-Speed-VoltageBoost-Technology/dp/B07QXV6N1B/ref=sr_1_1_sspa?crid=53ULGW8ZNDOW&keywords=power+bank&qid=1689699045&s=electronics&sprefix=power+bank%2Celectronics%2C144&sr=1-1-spons&sp_csd=d2lkZ2V0TmFtZT1zcF9hdGY&psc=1) |
| HC-SR04 sensors (5 pcs)	 | Used for the difstance calulations of objects and obstacles that are unwanted. | $8.99 | [Link](https://www.amazon.com/Organizer-Ultrasonic-Distance-MEGA2560-ElecRight/dp/B07RGB4W8V/ref=sr_1_2?crid=UYI359LWAAVU&keywords=hc+sr04+ultrasonic+sensor+3+pc&qid=1689699122&s=electronics&sprefix=hc+sr04+ultrasonic+sensor+3+pc%2Celectronics%2C123&sr=1-2) |
| HDMI to micro HDMI cable	 | Connecting the Rasberri pi 4 to the laptop to show on OBS video stream | $8.99 | [Link](https://www.amazon.com/UGREEN-Adapter-Ethernet-Compatible-Raspberry/dp/B06WWQ7KLV/ref=sr_1_5?crid=3S06RDX7B1X4O&keywords=hdmi+to+micro+hdmi&qid=1689699482&s=electronics&sprefix=hdmi+to+micro%2Celectronics%2C132&sr=1-5) |
| Video Capture card | Nessecary to display onto laptops | $16.99 | [Link](https://www.amazon.com/Capture-Streaming-Broadcasting-Conference-Teaching/dp/B09FLN63B3/ref=sr_1_3?crid=19YSORXLTIALH&keywords=video+capture+card&qid=1689699799&s=electronics&sprefix=video+capture+car%2Celectronics%2C140&sr=1-3) |
| SD card reader | Nessecary to flash microSD and install an os onto it | $4.99 | [Link](https://www.amazon.com/Reader-Adapter-Camera-Memory-Wansurs/dp/B0B9QZ4W4Y/ref=sr_1_4?crid=F124KSQOC5SO&keywords=sd+card+reader&qid=1689869007&sprefix=sd+card+reader%2Caps%2C126&sr=8-4) |
| Wired Mouse and Keyboard | Seperate mouse and keyboard needed to operate the Raberri Pi 4 | $25.99 | [Link](https://www.amazon.com/Wireless-Keyboard-Trueque-Cordless-Computer/dp/B09J4RQFK7/ref=sr_1_1_sspa?crid=2R048HRMFBA7Z&keywords=mouse+and+keyboard+wireless&qid=1689871090&sprefix=mouse+and+keyboard+wireless+%2Caps%2C131&sr=8-1-spons&sp_csd=d2lkZ2V0TmFtZT1zcF9hdGY&psc=1) |
| Basic connections components kit | Includes nesscary components such as the male to male and female to female as well as the male to female jumper wires, resistors, and LED'S. | $11.47 | [Link](https://www.amazon.com/Smraza-Breadboard-Resistors-Mega2560-Raspberry/dp/B01HRR7EBG/ref=sr_1_16?crid=27G99F3EADUCG&keywords=breadboard+1+pc&qid=1689894556&sprefix=breadboard+1+p%2Caps%2C185&sr=8-16) |
| Soldering Kit |Soldering Kit is used for the motor connections. | $13.60 | [Link](https://www.amazon.com/Soldering-Interchangeable-Adjustable-Temperature-Enthusiast/dp/B087767KNW/ref=sr_1_5?crid=1QYWI5SBQAPH0&keywords=soldering+kit&qid=1689900771&sprefix=soldering+kit%2Caps%2C169&sr=8-5) |

# Starter Project: Retro Arcade
<iframe width="560" height="315" src="https://www.youtube.com/embed/BF2v-AP0EPY?si=9xm23H1ms8oinXFz" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

Summary: 

My starter project was the retro arcade console. How it works is basically by receiving input by all the buttons on the front display which get processed through the integrated circuits which displays the game on a LED display. It also includes a buzzer for sound effects as well as runs on batteries. After I soldered all the electronic components onto the circuit board, I could play games such as Tetris and Snake with the system helping keep track of the player's score. This project taught me how to solder electronics to circuit boards, identify electronic components, and troubleshoot electrical connections. Making the Retro Arcade Console helped prepare me a lot for my main project, the Ball Tracking Robot.

Components Used:
-Microcontroller 
-Small LCD Screen
-Buzzer
-Capacitors
-4 x AA Battery holder/ AA batteries
-PCB (Circuit Board) 
-Solder Kit
-Header Pins
-Acrylic Case
-Variety Of Colored Buttons

Challenges Faced:

There were some challenged that were faced some harder than others. My first challenge was making solders to the board as everytime I soldered the solder wire was always getting attachted to other solder holes which could cause a short circuit and damge the board. There was also the problem of getting the red and black battery wires to attach neatly in these two tiny holes and hold them so I could solder them properly. I had to unsolder many parts multiple times as the retro arcade console simply was not turning on. But even after all these hardships I managed to fix every one of them and get the console working properly. 

# Other Resources/Examples
One of the best parts about Github is that you can view how other people set up their own work.
- [CLAUDE AI](https://claude.ai/share/d7c5e0e6-0105-45f7-9baa-1b5a52f87b24)
- [PORTFOLIO HELP](https://deringur.github.io/BSE_Derin_Portfolio/)
- [RASBERRY PI 4 MODEL B PIN LAYOUT](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcSo_bcd9R6ZhFa1BB1bqjqTkYidBfHI8zQddPhKNSacHQ&s)

