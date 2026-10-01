# Unit 2 - Simple Spinning 2D Scanner

## Goal

Our goal was to create a minimum testable prototype of a spinning 2D scanner that could eventually contribute to a more complex 3D scanner. We used a stepper motor to rotate an ultrasonic distance sensor so that it could measure the distance of objects from different directions.

## Process

Michelle, Cecilia, and I all contributed to writing and testing the Arduino code, while Michelle and Cecilia did most of the physical assembly of the scanner.

Our original code made the stepper motor rotate approximately 180° and then return to its starting position. The ultrasonic sensor was also able to measure distance.

While I was trying to create an angle vs. distance graph in Excel, I noticed a problem with our code. The original sequence was:

`rotate → rotate back → measure distance`

This meant that even though the sensor was rotating, it only measured the distance after returning to its starting position. Because of this, I did not actually have distance measurements from different angles to use for the graph.

## Code Development

### Original Code

This was the original code our group worked on:

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

### Figuring Out the Angle

To make the graph, I needed an angle to go with each distance measurement. Our code defines one rotation as 1025 steps:

```cpp
const int stepsPerRotation = 1025;
```

I used the number of steps to estimate the angle:

`angle = (steps moved / 1025) × 360°`

Instead of moving approximately 180° all at once, I changed the motor to move in smaller increments. About 10° is around 28 steps, so I used:

```cpp
myStepper.step(28);
```

This gave the sensor a chance to measure the distance at multiple points during the rotation.

### Collecting More Data

I added a loop to go from 0° to 180° in approximately 10° intervals:

```cpp
for (int angle = 0; angle <= 180; angle += 10)
```

I also changed the Serial Monitor output so that it printed the angle and distance together:

```cpp
Serial.print(angle);
Serial.print(",");
Serial.println(distance);
```

Instead of only getting a distance, the data could now look like:

```text
0,35.4
10,34.8
20,31.2
30,27.5
```

The first number is the angle and the second is the distance in centimeters.

I also added a second loop for the rotation back from 180° to 0°:

```cpp
for (int angle = 180; angle >= 0; angle -= 10)
```

and reversed the direction of the motor with:

```cpp
myStepper.step(-28);
```

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

## Graphing the Data

After changing the code, I organized the Serial Monitor data in Excel using one column for angle and another for distance. I then made an XY scatter plot with angle on the x-axis and distance on the y-axis.

Working on the graph was what helped me notice the problem with the original code. Watching the scanner move made it seem like it was working, but once I tried to graph the results, I realized that we needed to record the angle at the same time as each distance measurement.

### Data and Graph

I used Excel to organize the angle and distance measurements, calculate the X and Y coordinates, and create a 2D visualization of the scan.

[View my Excel data and graph](files/Scanner.xlsx)

## Reflection

All three of us worked on writing and testing the code, while Michelle and Cecilia did most of the physical assembly. I focused more on the angle and distance data and modifying the code once I noticed the problem while making the graph.

This process helped me understand that getting the motor and sensor to work separately is not enough to make a scanner. The movement of the motor has to match up with the sensor data so that each distance can be connected to a direction. Our next step is to continue testing the scanner and improve the accuracy of the measurements.
