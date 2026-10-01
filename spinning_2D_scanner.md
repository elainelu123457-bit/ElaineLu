# Unit 2 - Simple Spinning 2D Scanner

## Goal

Our goal was to create a **minimum testable prototype of a spinning 2D scanner** that could eventually contribute to a more complex 3D scanner. We used a stepper motor to rotate an ultrasonic distance sensor so that it could measure the distance of objects from different directions.

## Process

Michelle, Cecilia, and I all contributed to writing and testing the Arduino code, while Michelle and Cecilia did most of the physical assembly of the scanner.

Our original code made the stepper motor rotate approximately 180° and return to its starting position. The ultrasonic sensor also measured distance.

While I was working on creating an **angle vs. distance graph in Excel**, I noticed a problem with our code. Our original sequence was:

**rotate → rotate back → measure distance**

This meant that even though the sensor rotated, it only measured distance after returning to its starting position. Therefore, we were not actually collecting distance measurements at different angles.

> **Photo:** Add photo of assembled scanner here.

---

## Code Development

### 1. Original Code

This was the original code that our group worked on:

```cpp
#include <Stepper.h>

//Input pins
#define OUTPUT1   7                
#define OUTPUT2   6                
#define OUTPUT3   5              
#define OUTPUT4   4              

// steps per rotation
const int stepsPerRotation = 1025;

Stepper myStepper(stepsPerRotation, OUTPUT1, OUTPUT3, OUTPUT2, OUTPUT4);  

const int trigPin = 9;
const int echoPin = 10;

float duration, distance;

void setup() {
  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT);
  Serial.begin(9600);

  // speed of the motor in RPM
  myStepper.setSpeed(10);
}

void loop() {
  // Turn halfway forward
  myStepper.step(stepsPerRotation / 2);
  delay(500);

  // Reverse direction
  myStepper.step(-stepsPerRotation / 2);
  delay(500);

  digitalWrite(trigPin, LOW);
  delayMicroseconds(1);
  digitalWrite(trigPin, HIGH);
  delayMicroseconds(1);
  digitalWrite(trigPin, LOW);

  duration = pulseIn(echoPin, HIGH);
  distance = (duration * .0343) / 2;

  Serial.print("Distance: ");
  Serial.println(distance);

  delay(1);
}
```

### 2. Connecting Motor Steps to Angle

To make the graph, I needed both **distance and angle**. Our code defines one rotation as:

```cpp
const int stepsPerRotation = 1025;
```

I used the number of steps to estimate the angle:

**Angle = (steps moved / 1025) × 360°**

Instead of moving approximately 180° all at once, I changed the motor to move in smaller increments.

Approximately 10° is about 28 steps:

```cpp
myStepper.step(28);
```

This allowed the sensor to take measurements throughout the rotation instead of only after the full movement.

### 3. Collecting Measurements at Different Angles

I added a loop that tracks the angle from 0° to 180°:

```cpp
for (int angle = 0; angle <= 180; angle += 10)
```

This lets the program collect measurements at approximately:

**0°, 10°, 20°, 30° ... 180°**

### 4. Recording Angle and Distance Together

I changed the Serial Monitor output so that each distance measurement is paired with an angle:

```cpp
Serial.print(angle);
Serial.print(",");
Serial.println(distance);
```

The data is then formatted like:

```text
0,distance
10,distance
20,distance
30,distance
...
180,distance
```

This made it easier to transfer the data into Excel.

### 5. Adding the Reverse Scan

I also added another loop so the sensor could collect measurements while rotating from 180° back toward 0°:

```cpp
for (int angle = 180; angle >= 0; angle -= 10)
```

The motor moves in the opposite direction using:

```cpp
myStepper.step(-28);
```

---

## Final Code

```cpp
#include <Stepper.h>

//Input pins
#define OUTPUT1   7                
#define OUTPUT2   6                
#define OUTPUT3   5              
#define OUTPUT4   4              

// steps per rotation
const int stepsPerRotation = 1025;

Stepper myStepper(stepsPerRotation, OUTPUT1, OUTPUT3, OUTPUT2, OUTPUT4);  

const int trigPin = 9;
const int echoPin = 10;

float duration, distance;

void setup() {
  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT);
  Serial.begin(9600);

  // speed of the motor in RPM
  myStepper.setSpeed(10);
}

void loop() {

  for (int angle = 0; angle <= 180; angle += 10) {

    digitalWrite(trigPin, LOW);
    delayMicroseconds(2);
    digitalWrite(trigPin, HIGH);
    delayMicroseconds(10);
    digitalWrite(trigPin, LOW);

    duration = pulseIn(echoPin, HIGH);
    distance = (duration * .0343) / 2;

    Serial.print(angle);
    Serial.print(",");
    Serial.println(distance);

    if (angle < 180) {
      myStepper.step(28);
      delay(200);
    }
  }

  for (int angle = 180; angle >= 0; angle -= 10) {

    digitalWrite(trigPin, LOW);
    delayMicroseconds(2);
    digitalWrite(trigPin, HIGH);
    delayMicroseconds(10);
    digitalWrite(trigPin, LOW);

    duration = pulseIn(echoPin, HIGH);
    distance = (duration * .0343) / 2;

    Serial.print(angle);
    Serial.print(",");
    Serial.println(distance);

    if (angle > 0) {
      myStepper.step(-28);
      delay(200);
    }
  }
}
```

> **Photo:** Add screenshot of Serial Monitor showing angle and distance data here.

---

## Creating the Graph

I copied the Serial Monitor data into Excel and organized it into two columns:

| Angle (°) | Distance (cm) |
| --- | --- |
| 0 | measurement |
| 10 | measurement |
| 20 | measurement |
| 30 | measurement |
| ... | ... |
| 180 | measurement |

I then created an **XY scatter plot** with:

- **X-axis:** Angle (degrees)
- **Y-axis:** Distance (cm)

Creating the graph also helped me debug our code. I realized that simply rotating the sensor was not enough for a 2D scanner. We also needed to know **the angle at which each distance measurement was taken**.

> **Photo:** Add screenshot of Excel graph here.

---

## Reflection

All three of us contributed to writing and testing the code, while Michelle and Cecilia did most of the physical assembly. I focused more on creating the graph and modifying the code after noticing the problem with our original measurements.

This project helped me understand how the **stepper motor, ultrasonic sensor, code, and data visualization work together**. Trying to graph our results was especially useful because it helped me find a problem that was not obvious just from watching the scanner move.

Our next step is to test the scanner with objects in different positions and continue improving the accuracy of the angle and distance measurements.
