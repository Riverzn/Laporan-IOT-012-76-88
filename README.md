# IOT-Report-012-76-88

### Project Report: Distance Meter Using Raspberry Pi and HC-SR04 Ultrasonic Sensor

-----

### Group Members:

| Name                       | Student ID |
| -------------------------- | ---------- |
| Adiwidya Budi Pratama      | 5027241012 |
| Dimas Muhammad Putra       | 5027241076 |
| I Gede Bagus Saka Sinatrya | 5027241088 |

-----

## 1\. Introduction

### 1.1. Background

The Internet of Things (IoT) and embedded systems have become essential in the development of modern technology. The Raspberry Pi, as a single-board computer, offers a powerful yet affordable platform for learning and developing various electronics and programming projects. A fundamental application often built is distance measurement, which serves as a foundation for more complex projects such as obstacle-avoiding robots, automated parking systems, or assistive devices for the visually impaired.

This project aims to implement a simple distance measurement system using a Raspberry Pi connected to an HC-SR04 ultrasonic distance sensor. This sensor operates on the principle of sonar, sending out ultrasonic sound waves and measuring the time it takes for the waves to return after bouncing off an object.

### 1.2. Project Objectives

The objectives of this project are as follows:

  - To learn how to connect external components (sensors) to the Raspberry Pi's GPIO (General Purpose Input/Output) pins.
  - To understand the working principle of the HC-SR04 ultrasonic distance sensor.
  - To implement the Python programming language to read and process data from the sensor.
  - To create a program capable of measuring and displaying the distance of an object in real-time.

-----

## 2\. Tools and Materials

### 2.1. Hardware

  - **Raspberry Pi**: Any model with a GPIO header (e.g., Raspberry Pi 3 Model B+, Raspberry Pi 4).
  - **HC-SR04 Ultrasonic Distance Sensor**: The main component for measuring distance.
  - **Breadboard**: For creating the circuit without soldering.
  - **Jumper Wires (Male-to-Female)**: 4 pieces to connect the sensor to the Raspberry Pi.

### 2.2. Software

  - **Operating System**: Raspberry Pi OS (formerly Raspbian).
  - **Programming Language**: Python 3.
  - **Python Libraries**: `RPi.GPIO` for GPIO pin control and `time` for time-related functions.

-----

## 3\. Circuit and Schematic

The components are connected according to the following scheme. This project uses the BCM pin numbering mode, which refers to the GPIO numbers, not the physical pin numbers.

| HC-SR04 Sensor Pin | Raspberry Pi Pin (BCM) | Raspberry Pi Physical Pin | Description                           |
| ------------------ | ---------------------- | ------------------------- | ------------------------------------- |
| VCC                | 5V                     | Pin 2 or 4                | 5V power supply for the sensor        |
| Trig (Trigger)     | GPIO 23                | Pin 16                    | (Output) Sends the ultrasonic pulse   |
| Echo (Receive)     | GPIO 24                | Pin 18                    | (Input) Receives the reflected pulse  |
| GND (Ground)       | Ground                 | Pin 6, 9, 14, etc.        | Connects to the common ground         |

-----

## 4\. Python Code

The following Python script reads the sensor data and sends it to a ThingSpeak channel.

```python
import RPi.GPIO as GPIO
import time
import requests  # To send data to ThingSpeak

# Use BCM GPIO numbering
GPIO.setmode(GPIO.BCM)

# Set GPIO pin numbers
TRIG = 23
ECHO = 24

# Set up GPIO pins
GPIO.setup(TRIG, GPIO.OUT)
GPIO.setup(ECHO, GPIO.IN)

# ThingSpeak API Key
API_KEY = "SGLTSWIQ88VBQM4Y"
THINGSPEAK_URL = "https://api.thingspeak.com/update"

def measure_distance():
    """Measures distance using the ultrasonic sensor."""
    # Ensure trigger is low
    GPIO.output(TRIG, False)
    time.sleep(0.5)

    # Send a 10us pulse to trigger
    GPIO.output(TRIG, True)
    time.sleep(0.00001)
    GPIO.output(TRIG, False)

    # Measure the time the echo pin is high
    while GPIO.input(ECHO) == 0:
        start_time = time.time()

    while GPIO.input(ECHO) == 1:
        end_time = time.time()

    # Calculate distance
    duration = end_time - start_time
    # Speed of sound is 34300 cm/s. Divide by 2 for the round trip.
    distance = duration * 17150
    distance = round(distance, 2)
    
    return distance

try:
    while True:
        distance = measure_distance()
        print(f"Distance: {distance} cm")

        # Send data to ThingSpeak (using field1)
        payload = {'api_key': API_KEY, 'field1': distance}
        try:
            response = requests.get(THINGSPEAK_URL, params=payload)

            # Check if the request was successful
            if response.status_code == 200:
                print("Data sent to ThingSpeak successfully!")
            else:
                print(f"Failed to send data. Status code: {response.status_code}")
        except requests.exceptions.RequestException as e:
            print(f"An error occurred: {e}")

        # ThingSpeak has a rate limit of about 15 seconds per update
        time.sleep(15)

except KeyboardInterrupt:
    print("\nProgram stopped.")
    GPIO.cleanup()
```

-----

## 5\. How to Run the Program

1.  Ensure all circuit connections are correct.
2.  Open the Terminal application on your Raspberry Pi.
3.  Navigate to the directory where you saved the `measure_distance.py` file.
4.  Run the program by typing the following command and pressing Enter:
    ```bash
    python3 measure_distance.py
    ```
5.  Point the sensor at various objects at different distances to see the output.
6.  To stop the program, press **Ctrl + C** on the keyboard.

-----

## 6\. Program Results and Output

When the program is running, the terminal will display real-time distance readings every 15 seconds. Simultaneously, this data will be uploaded to the configured ThingSpeak channel, allowing for remote monitoring and data logging.

**Example Terminal Output:**

```
Distance: 45.21 cm
Data sent to ThingSpeak successfully!
Distance: 102.88 cm
Data sent to ThingSpeak successfully!
Distance: 15.63 cm
Data sent to ThingSpeak successfully!
```

-----

## 7\. Conclusion

This project successfully implemented a distance measurement system using a Raspberry Pi and an HC-SR04 ultrasonic sensor. Through this project, we learned about GPIO pin functionality, the principles of ultrasonic sensors, and the implementation of Python code to process sensor data. The system performed well, providing accurate real-time distance readings suitable for various basic applications.

This project can be further developed by adding an LCD screen to display the distance, an actuator like a buzzer for an alarm system, or integrating it into a larger robotics system.
