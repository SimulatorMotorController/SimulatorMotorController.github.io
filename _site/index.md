# SMC3 
The SMC3 is a "Simulator Motor Controller for 3 Motors" written for the Arduino UNO R3.
At the time of writing this it has not been tested on any other Arduino model. The SMC3
uses a PID motor control loop. The PID algorithm has been optimised for this application
and achieves 4096 PID updates per second for all three motors giving smooth precise motor
control with well tuned parameters. Some characteristics of the controller:

- Designed specifically for use with Simtools motion control software
- Controls the position of upto three motors using analogue feedback
- Hard (reverse) braking if motors move beyond defined limits
- 10bit accuracy (1024 steps) for both "target" position commands and position "feedback"
- Extensive command set to setup and monitor the parameters for each motor
- A second serial port for (optional) real time "live" monitoring/configuration
- 4096 PID calculation updates per second for each motor
- Companion windows software to configure and monitor the SMC3
- There are currently two MODES of operation that are configured by a single line of
  code at compile time before uploading to the Arduino.

### MODE1: Supports the more common H-Bridges used in the forums.

This mode has a PWM output pin plus two Motor Direction output pins. Examples include the MonsterMoto shield.

![MonsterMoto](/assets/images/MotoMonster.jpg)

### MODE2: Designed for H-Bridges that require direct drive of Highside and Lowside switch inputs.

In this mode one switch is driven as a direction pin and the other with the PWM output however the PWM 
duty needs to be inverted whenever the motor changes direction with the direction pin. (An alternate 
approach would be to switch the PWM between inputs as direction changes) An example H-Bridge that uses
this mode is the cheap 43A IBT-2 found on ebay.

![IBT-2](/assets/images/IBT-2.jpg)

## Software Setup

As the SMC3 is an Arduino motor controller the first thing you need to do is get the software onto your "Arduino UNO R3".
- [Download the latest SMC3 software](https://github.com/SimulatorMotorController/SMC3/archive/master.zip)
- Get the Arduino IDE tools installed on your computer if you don't already have them. Follow the [Arduino Getting Started](http://arduino.cc/en/Guide/Windows)
  guide (Note the step for installing the drivers!)
- Open the SMC3.ino with the Arduino IDE
- Edit the code at the top if the file to select the _MODE_ you will be using
- Compile and Download the SMC3.ino program to your Arduino UNO R3
- If you want, you could now test out the [Windows SMC3 Utilities](https://github.com/SimulatorMotorController/SMC3Utils)
  software and check communications before proceeding further.

## Wiring Details

                        +----+              +------+
                      +-|PWR |--------------| USB  |-----+
                      | |    |              |      |     |
                      | +----+              +------+    o|
                      |                                 o|
                      |                                 o|AREF
                      |                                 o|GND
                    NC|o                                o|13    TTL (software) Serial
                 IOREF|o  +----+                        o|12    TTL (software) Serial
                 RESET|o  |    |                        o|11~  PWM Motor 3
                  3.3V|o  |    |                        o|10~  PWM Motor 2
                    5V|o  |    |    Arduino             o|9~    PWM Motor 1
                   GND|o  |    |      UNO               o|8
                   GND|o  |    |                         |
                   VIN|o  |    |                        o|7      Motor 3 ENB
                      |   |    |                        o|6~    Motor 3 ENA
       Motor 1 Pot  A0|o  |    |                        o|5~    Motor 2 ENB
       Motor 2 Pot  A1|o  |    |                        o|4      Motor 2 ENA
       Motor 3 Pot  A2|o  |    |                        o|3~    Motor 1 ENB
                    A3|o  |    |                        o|2      Motor 1 ENA
                    A4|o  |    |                        o|1
                    A5|o  +-/\-+                        o|0
                      +-\                      /---------+
                         \--------------------/

![SMC3 Pinout](/assets/images/SMC3Pinout.jpg)


### MODE1

Example wiring details for Monster Motor Board (again thanks to @[eaorobbie](https://www.xsimulator.net/community/members/2210/) for drawing)


![MotoMonster Setup](/assets/images/MotoMonsterSetup.jpg)

### MODE2

Arduino to H-Bridge (Chinese 43A)

    ENA ----> IN1/RPWM
    ENB --+-> EN1/R_EN
    +-> EN2/L_EN (ie connect ENB to both R_EN and L_EN)
    PWM ----> IN2/L_PWM
    n.c. ----> IS1/R_IS
    n.c. ----> IS2/L_IS
    +5V ----> VCC
    GND ----> GND


The image below shows wiring for two motors... Thanks to @[eaorobbie](https://www.xsimulator.net/community/members/2210/) for getting me started with the image.


![Wiring 2 Motors](/assets/images/Wiring-2-Motors.jpg)


## Initial Setup

I strongly recommend you follow the steps below if this is the first time you are using SMC3 with your simulator to reduce the risk of damaging anything.

  1. Disconnect the motor power supply
  0. Make sure Simtools is not running – we’re not ready for that yet!
  0. Wire up the Arduino (with SMC3 installed) to your H-Bridges and connect to your computer via USB
  0. Run the Windows SMC3 Utility software and make sure it communicates with the Arduino (There is no need to set baud rates, they are not configurable)
  0. Set the Kp, Ki, Kd, PWMmin, PWMmax, PWMrev to 0 for ALL motors (This will make sure the motors don’t move)
  0. Set Clip to 255 (you need to do this first) and Limit to 255 (This will give you plenty of margin if something goes wrong while setting up)
  0. Turn on the power to your motors – nothing should move at this stage!
  0. Set Kp to about 400
  0. Now slowly, increase PWMmax… at some point the motor should start to move. When it does check the “Green” feedback line is moving toward the “Blue” target position.
      - If it is then that motor and feedback is wired correctly - proceed to test other motors.
      - If it is moving away turn off motor power immediately (or quickly reduce PWMmax again). In this case you need to either reverse the wires to the motor being tested –OR– reverse the +5V and GND wires to your feedback pot for the motor being tested (do not do both). Restart the test from the beginning.
  0. Do the above for each motor

## The Second Serial Port

Temporarily disabled as of v0.63 - Use UDP Mode with SMC3Utils instead.


## Simtools Setup

Setup for **Simtools 2** is very straight forward:

`[A<Axis1a>][B<Axis2a>][C<Axis3a>]`

Set for `10 bit binary` output

Baud rate `500000 , N , 8 , 1` (10ms delay)


## Potentiometer Scaling

Potentiometer Scaling (POT scaling) is an input that can be used to scale the motion output based on the POT position.

The non linear scaling is designed to try and reduce the overall motion but maintain the "smaller" movements as best 
possible - ie road noise and stuff, but scale back on the huge side to side motion you get in higher power cars.

- POT scaling can be setup as one of two modes - Linear or non-linear
- **DO NOT ENABLE POT SCALING UNLESS POT ATTACHED TO ARDUINO**
- AN5 used for POT scaling input

![POT scaling](/assets/images/scaling.jpg)


