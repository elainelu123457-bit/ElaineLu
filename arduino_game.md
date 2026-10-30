# Arduino + PySerial: Mini Trombone Champ

## 1. My Plan

For this project, I designed a simplified version of *Trombone Champ*, a rhythm game where players adjust their musical pitch to match notes moving across the screen.

I chose the **HC-SR04 ultrasonic distance sensor** because it allows players to control the game through hand movements instead of a keyboard. Moving a hand closer to or farther from the sensor changes the musical pitch.

### Materials
- Arduino Uno
- HC-SR04 ultrasonic distance sensor
- Breadboard
- 4 jumper wires
- USB cable
- Laptop

### Software
- **Arduino IDE:** Reads sensor measurements.
- **PySerial:** Transfers data from Arduino to Python.
- **PyGame:** Displays the game and processes inputs.
- **NumPy:** Generates musical sounds.

### Gameplay

The game is designed to display notes scrolling horizontally across the screen while the player moves their hand in front of the distance sensor.

The sensor measurement determines the player's musical pitch. Matching the correct pitch when a note reaches the scoring line earns points.

---

## 2. My Process

### Step 1: Arduino and Distance Sensor

I designed the sensor setup using the following connections:

| HC-SR04 | Arduino |
|---|---|
| VCC | 5V |
| GND | GND |
| TRIG | Pin 9 |
| ECHO | Pin 10 |

The Arduino code sends an ultrasonic pulse, measures how long the echo takes to return, and calculates the distance in centimeters. The distance is then transmitted through serial communication at 9600 baud.

**AI Usage:** I used ChatGPT to generate the initial Arduino code and understand how the trigger pin, echo pin, and distance calculation work.

#### Arduino Code

```cpp
const int trig = 9;
const int echo = 10;

void setup() {
  Serial.begin(9600);
  pinMode(trig, OUTPUT);
  pinMode(echo, INPUT);
}

void loop() {
  digitalWrite(trig, LOW);
  delayMicroseconds(2);

  digitalWrite(trig, HIGH);
  delayMicroseconds(10);
  digitalWrite(trig, LOW);

  long duration = pulseIn(echo, HIGH, 25000);

  if (duration > 0) {
    float distance = duration * 0.0343 / 2;
    Serial.println(distance);
  }

  delay(30);
}
```

### Step 2: PySerial Communication

Next, I developed the Python serial communication code to receive distance measurements from the Arduino.

The Arduino transmits distance values through USB, while Python reads each line and converts it into usable data.

**AI Usage:** ChatGPT helped me write the PySerial code and understand how `serial.Serial()`, `readline()`, and `decode()` work.

#### Python Serial Code

```python
import serial

arduino = serial.Serial('/dev/cu.usbmodem1101', 9600)

while True:
    distance = arduino.readline().decode().strip()
    print(distance)
```

The serial port needs to match the Arduino's actual connection.

### Step 3: Converting Distance into Musical Notes

I designed a system that divides the sensor's distance range into seven musical notes.

| Distance | Musical Note |
|---|---|
| 5–8 cm | C |
| 8–11 cm | D |
| 11–14 cm | E |
| 14–17 cm | F |
| 17–20 cm | G |
| 20–23 cm | A |
| 23–26 cm | B |

Each distance range corresponds to a musical note, allowing hand movements to control pitch.

**AI Usage:** I used ChatGPT to help develop the distance-to-note conversion function and understand how conditional statements could turn sensor measurements into game inputs.

#### Distance Conversion Code

```python
def distance_to_note(distance):
    if distance < 8:
        return "C"
    elif distance < 11:
        return "D"
    elif distance < 14:
        return "E"
    elif distance < 17:
        return "F"
    elif distance < 20:
        return "G"
    elif distance < 23:
        return "A"
    else:
        return "B"
```

### Step 4: PyGame Integration

I designed a PyGame interface featuring:

- Horizontally scrolling notes
- A pitch indicator controlled by hand distance
- Musical sounds corresponding to each note
- A scoring system awarding 100 points for correct notes
- A combo counter for consecutive correct notes

I selected *Twinkle Twinkle Little Star* as the first song because its simple melody makes the game easier to develop and test.

The Python program compares the player's current pitch with the target note when it reaches the scoring line.

**AI Usage:** I used ChatGPT to generate the initial PyGame code, including scrolling notes, musical tones, and scoring. I reviewed the code to understand how sensor readings influence gameplay.

---

## 3. How I Used AI Tools

I used ChatGPT throughout the planning and coding process, primarily to help generate code and understand how it worked.

- **Arduino Code:** AI helped generate code for measuring distance with the HC-SR04 sensor and transmitting measurements through serial communication. I reviewed how the calculations and functions worked.

- **PySerial:** AI helped write Python code for receiving serial data and explained how Arduino's output could be processed in Python.

- **PyGame:** AI generated the initial game implementation, including moving notes, musical sounds, and scoring. I reviewed how the different components interacted.

- **Understanding the Code:** I used AI explanations to better understand how Arduino and Python communicate and how physical measurements can be converted into game inputs.

AI was especially useful for developing the initial code and explaining unfamiliar functions. Reviewing the generated code helped me understand the relationship between hardware inputs, serial communication, and game mechanics.

---

## 4. My Result

I developed the code and gameplay design for a simplified version of *Trombone Champ* using Arduino, PySerial, and PyGame.

### Game Features

- **Distance-controlled pitch:** Hand movements determine musical notes.
- **Scrolling notes:** Players match their pitch to notes moving across the screen.
- **Sound generation:** Python generates a different tone for each note.
- **Scoring:** Correct notes earn 100 points.
- **Combo system:** Consecutive correct notes increase the combo.
- **Keyboard controls:** Number keys 1–7 provide an alternative way to control pitch without an Arduino.

### Gameplay Sequence

1. The player moves their hand in front of the ultrasonic distance sensor.
2. The Arduino measures the distance and sends it to Python.
3. Python converts the measurement into a musical note.
4. PyGame displays the player's pitch and plays the corresponding sound.
5. The player matches incoming notes to earn points.

### Testing and Improvements

The game code includes keyboard controls that allow the gameplay to be tested without an Arduino. The sensor-controlled version still requires hardware testing.

One potential challenge is that ultrasonic sensor measurements may fluctuate, causing unintended pitch changes. Serial communication could also introduce small delays that affect the timing of the rhythm game.

To improve the project, I would experiment with smoothing sensor readings and adjusting the timing of the notes.

---

## 5. Reflection

This project helped me understand how Arduino and Python can work together to create interactive experiences. Instead of using a sensor only to display measurements, I explored how its data could become meaningful game inputs.

The most interesting part was connecting physical hand movements to musical pitch. I also learned more about serial communication and how Python can process information from external hardware.

Using AI to generate and explain code helped me understand unfamiliar functions and how the different parts of the program connect.

If I continued developing this project, I would add smoother pitch transitions, more songs, and improved sound effects to make the game more similar to *Trombone Champ*.
