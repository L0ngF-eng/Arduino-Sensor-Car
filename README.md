# Arduino Obstacle-Avoiding Robot

A simple obstacle-avoiding robot built with an Arduino, two 28BYJ-48 stepper motors, and an HC-SR04 ultrasonic sensor. The robot drives forward, detects walls, reverses 13 cm back, and turns in a random direction to navigate around obstacles.

---

## Hardware

| Component | Quantity |
|-----------|----------|
| Arduino Uno (or compatible) | 1 |
| 28BYJ-48 Stepper Motor + ULN2003 driver | 2 |
| HC-SR04 Ultrasonic Sensor | 1 |
| Chassis / robot frame | 1 |
| Power supply (battery pack) | 1 |
| Jumper wires | — |

---

## Wiring

### Ultrasonic Sensor (HC-SR04)
| Sensor Pin | Arduino Pin |
|------------|-------------|
| VCC | 5V |
| GND | GND |
| Trig | 2 |
| Echo | 3 |

### Left Motor (ULN2003 driver)
| Driver Pin | Arduino Pin |
|------------|-------------|
| IN1 | 8 |
| IN2 | 10 |
| IN3 | 9 |
| IN4 | 11 |

### Right Motor (ULN2003 driver)
| Driver Pin | Arduino Pin |
|------------|-------------|
| IN1 | 4 |
| IN2 | 6 |
| IN3 | 5 |
| IN4 | 7 |

---

## How It Works

1. **Forward** — the robot drives forward continuously while no obstacle is detected within 15 cm.
2. **Wall detected** — if the sensor reads ≤ 15 cm, the robot stops and reverses until the obstacle clears.
3. **Random turn** — once clear, it randomly turns left or right (700 steps) before resuming forward motion.

---

## Settings

These constants can be adjusted at the top of the sketch:

| Constant | Default | Description |
|----------|---------|-------------|
| `motorSpeed` | `12` | RPM for both stepper motors |
| `wallDistance` | `15` | Obstacle detection threshold in cm |
| `stepsPerRevolution` | `2048` | Steps per full revolution (28BYJ-48) |

---

## Dependencies

- [Arduino Stepper Library](https://www.arduino.cc/reference/en/libraries/stepper/) — built into the Arduino IDE, no extra installation needed.

---

## Setup

1. Wire the components as shown above.
2. Open `robot.ino` in the Arduino IDE.
3. Select your board and port under **Tools**.
4. Click **Upload**.

---

## Notes

- The 28BYJ-48 is a slow motor (~12 RPM max reliable speed). If the robot moves sluggishly, this is expected — it's a trade-off for the torque and low cost of these motors.
- A `pulseIn` timeout of 25,000 µs (~25 ms) is set on the ultrasonic sensor. A return value of `0` means no echo was received and is treated as "no obstacle."
- Both motors step synchronously one step at a time, so turning is blocking — the sensor is not polled mid-turn.


## Code
```cpp
#include <Stepper.h>

 

const int stepsPerRevolution = 2048;

 

// Ultrasonic sensor

const int trigPin = 2;

const int echoPin = 3;

 

// Motors (correct pin order for 28BYJ-48)

Stepper leftMotor(stepsPerRevolution, 8, 10, 9, 11);

Stepper rightMotor(stepsPerRevolution, 4, 6, 5, 7);

 

// Settings

const int motorSpeed = 12;

const int wallDistance = 15; // cm

 

void setup() {

  pinMode(trigPin, OUTPUT);

  pinMode(echoPin, INPUT);

 

  leftMotor.setSpeed(motorSpeed);

  rightMotor.setSpeed(motorSpeed);

 

  randomSeed(analogRead(A0));

}

 

void loop() {

  int distance = getDistance();

 

  // Go forward if no wall

  if (distance > wallDistance || distance == 0) {

    forwardSteps(20);

  }

  // Wall detected

  else {

    stopCar();

    delay(300);

 

    // Reverse until wall disappears

    while (true) {

      distance = getDistance();

 

      if (distance > wallDistance || distance == 0) {

        break;

      }

 

      backwardSteps(20);

    }

 

    stopCar();

    delay(300);

 

    // Random turn

    if (random(0, 2) == 0) {

      turnLeftSteps(700);

    } else {

      turnRightSteps(700);

    }

 

    stopCar();

    delay(300);

  }

}

 

int getDistance() {

  digitalWrite(trigPin, LOW);

  delayMicroseconds(2);

 

  digitalWrite(trigPin, HIGH);

  delayMicroseconds(10);

  digitalWrite(trigPin, LOW);

 

  long duration = pulseIn(echoPin, HIGH, 25000);

 

  if (duration == 0) return 0;

 

  return duration * 0.034 / 2;

}

 

// FIXED DIRECTIONS (now forward is actually forward)

void forwardSteps(int steps) {

  for (int i = 0; i < steps; i++) {

    leftMotor.step(1);

    rightMotor.step(1);

  }

}

 

void backwardSteps(int steps) {

  for (int i = 0; i < steps; i++) {

    leftMotor.step(-1);

    rightMotor.step(-1);

  }

}

 

void turnLeftSteps(int steps) {

  for (int i = 0; i < steps; i++) {

    leftMotor.step(-1);

    rightMotor.step(1);

  }

}

 

void turnRightSteps(int steps) {

  for (int i = 0; i < steps; i++) {

    leftMotor.step(1);

    rightMotor.step(-1);

  }

}

 

void stopCar() {

  delay(100);

}
```
