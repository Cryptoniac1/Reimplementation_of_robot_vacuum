+++
draft = false
title = 'Showcase'
+++

![The Vacuum](/Reimplementation_of_robot_vacuum/IMG_20260312_225012_233.jpg)

# Why?

Have you ever looked at your robot vacuum and been disappointed because you
don't know what it's doing? Do you hate that to run your new robot vacuum you
need to have an always connected online account? Do you just want fun with an
old vacuum? This can fix 2 of those.

The Shark Matrix Robot vacuum is a great vacuum that has LIDAR, crash sensors,
self emptying technology, mapping functionality, and more. This is all great,
but once it starts to break down, it becomes useless, and easier to just
replace. Not to mention that when you get it new, you have to make an account to
use most of the functionality. Wouldn't it be so much better if you can control
it with a remote and define your area? Well that's exactly what this project
does.

# TOC

- [Hardware](/Reimplementation_of_robot_vacuum/showcase/#hardware)
  - [Robot Vacuum](/Reimplementation_of_robot_vacuum/showcase/#robot-vacuum)
  - [Texas Instrument CC3200](/Reimplementation_of_robot_vacuum/showcase/#texas-instrument-cc3200)
  - [Adafruit OLED](/Reimplementation_of_robot_vacuum/showcase/#adafruit-eyespi-oled)
  - [Remote](/Reimplementation_of_robot_vacuum/showcase/#remote)
  - [IR Receiver](/Reimplementation_of_robot_vacuum/showcase/#ir-receiver)
  - [Voltage Converters](/Reimplementation_of_robot_vacuum/showcase#voltage-converters)
  - [L289N H-Bridge](/Reimplementation_of_robot_vacuum/showcase#289n-h-bridge)
- [Architecture](/Reimplementation_of_robot_vacuum/showcase/#hardware)
  - [Electrical](/Reimplementation_of_robot_vacuum/showcase/#electrical)
    - [Pinout of CC3200](/Reimplementation_of_robot_vacuum/showcase/#pinout-used)
  - [Digital](/Reimplementation_of_robot_vacuum/showcase/#digital)
    - [Interrupts](/Reimplementation_of_robot_vacuum/showcase/#interrupts)
    - [AWS](/Reimplementation_of_robot_vacuum/showcase/#aws)
    - [Loop](/Reimplementation_of_robot_vacuum/showcase/#loop)
- [Demo](/Reimplementation_of_robot_vacuum/showcase/#demo-video)

# Hardware

## Robot Vacuum

![Shark Matrix Vacuum](/Reimplementation_of_robot_vacuum/masked_image.png)

The Shark Matrix Robot Vacuum was used for this project. The choice came down to
the fact that I was able to get an old one that was to be thrown away. It has
many sensors such as: LIDAR, Camera, Crash, and depth. Not all of these were
implemented as it was outside of the scope of the project.

## Texas Instrument CC3200

![CC3200](/Reimplementation_of_robot_vacuum/board.png)

The TI CC3200 is a board that has many built-in sensors, including an
accelerometer, 4 hardware timers, and over 15 addressable GPIO pins. It also has
built-in support for I2C, SPI, and many other protocols. This board was the
brains of the operation that controlled the motors, and the accelerometer was
used for detection of whether or not it was on an edge. To read the
accelerometer you must communicate with it using I2C. The pinout for this board
can be viewed [here](/Reimplementation_of_robot_vacuum/showcase/#pinout-used).

## Adafruit EYESPI OLED

![Adafruit OLED](/Reimplementation_of_robot_vacuum/1431-18.jpg)

To display information about the vacuum to the user the Adafruit EYESPI OLED was
used. This displays the current mode and displays how to control the vacuum. As
the name implies, it communicates with the CC3200 over SPI. Unfortunately during
testing the display used stopped functioning as it was old. As such you cannot
see it in the demo video.

## Remote

![Remote](/Reimplementation_of_robot_vacuum/masked_remote.png)

An AT&T universal remote was used to control the robot. It had tv code: 1156,
which corresponds to a Samsung TV code. To view how it controlled the program
see [digital](/Reimplementation_of_robot_vacuum/showcase/#digital).
## IR Receiver

![IR receiver](/Reimplementation_of_robot_vacuum/ir-module-package.png)

I used a TSOP311xx IR receiver to receive input from the
[remote](/Reimplementation_of_robot_vacuum/showcase#remote). The output was wired to pin
8 of the CC3200, and was given 5v input from the battery with a low pass filter
of a 100Ω resistor and a 100µF capacitor. Its signal was controlled via
interrupts.

## Voltage Converters

![The Voltage Converters](/Reimplementation_of_robot_vacuum/61OeeOgShLL._AC_SL1500_.jpg)

The voltage converters were connected to the original battery to supply the
correct voltage to the CC3200, and to the motors. One converter was used to
convert from the 30v battery to 20V for the motors, and one was used to convert
from 30V to 5V.

## L289N H-Bridge

![The Bridges](/Reimplementation_of_robot_vacuum/712PvtiaP9L._AC_SL1500_.jpg)

These bridges were used to control the motors. The original battery was
connected to the [voltage converters](/Reimplementation_of_robot_vacuum/showcase#voltage-converters) which were connected to these for the
20v to control the motors (it can take up to 35 without problem), and 5v for the
board to drive them.

# Architecture 

## Electrical

![System Architecture Diagram](/Reimplementation_of_robot_vacuum/grim_26-03-13-1773459561.png)

### Pinout used

The wiring of the board is as follows:

| Pin | Use                  | Connection      |
|-----|----------------------|-----------------|
| 1   | Accelerometer SCL    | Accelerometer   |
| 2   | Accelerometer SDA    | Accelerometer   |
| 3   | OLED Reset           | OLED pin 4      |
| 4   | Onboard Switch       |                 |
| 5   | OLED SCK             | OLED pin 2      |
| 6   | Left Motor Forward   | LN289 in 1      |
| 7   | OLED MOSI            | OLED pin 1      |
| 8   | IR receiver pin      | IR drain        |
| 15  | Onboard Switch       |                 |
| 18  | Left Motor Backward  | LN289 in 2      |
| 50  | OLED CS              | OLED pin 50     |
| 53  | Crash Switch         | (Unimplemented) |
| 55  | UART TX              | debugging       |
| 57  | UART RX              | debugging       |
| 58  | Right Motor Backward | LN289 in3       |
| 59  | Crash Switch         | (Unimplemented) |
| 60  | OLED DC              | OLED pin 3      |
| 61  | Right motor forward  | LN289 in4       |
| 64  | Cleaning Motor       | LN289 (2) in4   |

## Digital

### Interrupts

There were a variety of interrupts used. The first was a GPIO interrupt for the
IR receiver, a GPIO interrupt for the crash sensors (unimplemented), a GPIO
interrupt for the onboard pins, a timer interrupt for correcting the motors, and
finally a timer interrupt for reading the remote data.

####  GPIO Crash and Button Interrupts

The crash sensors that were built into the Robot Vacuum were unable to give
usable data within the time period allotted. As such in code they were
implemented but unused.

The onboard buttons, however, made use of an interrupt to set which button is
pressed. As shown in [loop](/Reimplementation_of_robot_vacuum/showcase/#loop),
almost everything can be done through buttons 1, and 2. As such, the left button
was set to be interpreted as a press to the "1" button on the remote. Similarly,
the right was set to be interpreted as a "2".

#### Remote Interrupts

The remote makes use of 2 interrupts. The first is a trigger to the falling edge
of the IR receiver, and the second is a timer. The timer will read how long
passes between each falling edge when a remote is pressed, and interprets the
data. A long perios of 1.6ms is interpreted as a 0, and a short period of 530µs
is interpreted as a 1. The chart for each keys code can be seen below.

|  Key  |     hex    |
|:-----:|:----------:|
|   0   | 0x1f1f7788 |
|   1   | 0x1f1fdf20 |
|   2   | 0x1f1f5fa0 |
|   3   | 0x1f1f9f60 |
|   4   | 0x1f1fef10 |
|   5   | 0x1f1f6f90 |
|   6   | 0x1f1faf50 |
|   7   | 0x1f1fcf30 |
|   8   | 0x1f1f4fb0 |
|   9   | 0x1f1f8f70 |
|  Del  | 0x1f1f07f8 |
| Enter | 0x1f1fe916 |
|  Mute | 0x1f1f0ff0 |
|  Last | 0x1f1f37c8 |

#### Motor Correction Interrupts

After initial disassembly of the vacuum, it was found that the 2 motors move at
different rates. While the L289N does support control over PWM, it slowed the
motors further than it should of. As such much testing was done to ensure the
right motor would turn off for the right amount of time to continue moving
straight. After much testing it was determined that the motor should be turned
off every quarter second, and turned back on after 35ms.

### AWS

Just because there is no account anymore, doesn't mean you cannot store a map.
Amazon Web Services IOT Core was used to store the map and state of the device.
This allows for not only a stored map, but also remote triggering via AWS.

### Loop

![Loop](/Reimplementation_of_robot_vacuum/grim_26-03-13-1773461811.png)

At the beginning of the loop, it will check if 15s has passed, if it has it will
call AWS to get the map and the desired state.

#### Mapping Mode

In mapping mode, you an control the robot with the remote or onboard buttons to
define your map area. The controls are as follows:

| Key   | Action                                      |
|-------|---------------------------------------------|
| 1     | Turn Left 90 Degrees                        |
| 2     | Move Forward 1 Foot                         |
| 3     | Turn Right 90 Degrees                       |
| 5     | Move Backward 1 Foot                        |
| Enter | Save map to AWS and Return to Original Spot |

As an additional feature, if the robot moves on a ledge and the accelerometer's
Z-axis moves too much, it will go into panic and stop in place.

You can see mapping mode in [demo](/Reimplementation_of_robot_vacuum/showcase/#demo-video).


#### Cleaning Mode

Upon entering cleaning mode, the vacuum will snake through its defined map,
visiting every part of the array to clean the ground. It will simply start at an
extreme corner of the area, and snake through the area. 

You can see cleaning mode in [demo](/Reimplementation_of_robot_vacuum/showcase/#demo-video).

# Demo Video

<figure>
  <video controls width="400">
    <source src="/Reimplementation_of_robot_vacuum/part1.mp4" type="video/mp4">
  </video>
  <figcaption>Demo Showing Mapping Mode</figcaption>
</figure>

<figure>
  <video controls width="400">
    <source src="/Reimplementation_of_robot_vacuum/part2.mp4" type="video/mp4">
  </video>
  <figcaption>Demo an already active map</figcaption>
</figure>
