# Laboratory Activity 6 – Basic Actuator Control

**Course:** BCA152 – Microcontrollers
**Laboratory Activity:** No. 6 – Basic Actuator Control
**Platform:** ESP32 Dev Module
**Framework:** Arduino
**Development Environment:** Visual Studio Code + PlatformIO

---

## 1. Overview

This laboratory activity demonstrates basic actuator control using an ESP32 microcontroller and a motor driver interface.

The project implements the Example 7 control logic for a button-controlled motor. A push button is used as the user input, while two ESP32 GPIO pins provide the motor driver's AIN1 and AIN2 control signals. The driver's nSLEEP input is also controlled by the ESP32 so that the driver can be enabled after initialization.

The program uses the Arduino framework and is organized as a PlatformIO project.

---

## 2. Objectives

The objectives of this laboratory activity are to:

* Build and understand a basic motor-driver control circuit.
* Identify the relevant control signals of a motor-driver breakout.
* Control a motor using a push button and an ESP32.
* Understand the roles of AIN1, AIN2, nSLEEP, VM, and common ground.
* Demonstrate the motor driver's run and inactive/coast control states.
* Prepare the project for optional PWM-based motor-speed control.

---

## 3. Hardware

The project is designed around the following functional components:

* ESP32 development board
* DC motor
* Motor-driver breakout
* Push button
* External motor power supply
* Connecting wires
* Breadboard

The exact motor rated voltage, stall current, and motor-driver breakout model should be taken from the datasheets of the actual hardware used in the laboratory setup.

---

## 4. ESP32 Pin Assignment

| Function            | ESP32 GPIO |
| ------------------- | ---------: |
| Push Button         |    GPIO 23 |
| Motor Driver AIN1   |    GPIO 21 |
| Motor Driver AIN2   |    GPIO 22 |
| Motor Driver nSLEEP |    GPIO 27 |

The pin assignments are defined directly in `src/main.cpp`.

---

## 5. Control Logic

The push button is configured using the ESP32's internal pull-up resistor:

```cpp
pinMode(BUTTON_PIN, INPUT_PULLUP);
```

Because of the pull-up configuration:

* Button released → GPIO 23 reads `HIGH`
* Button pressed → GPIO 23 reads `LOW`

The motor-control logic continuously checks the button state.

When the button is pressed:

```text
AIN1 = HIGH
AIN2 = LOW
```

When the button is released:

```text
AIN1 = LOW
AIN2 = LOW
```

This provides the two control states required by the Example 7 button-controlled motor operation.

---

## 6. Motor Driver Signals

### AIN1

AIN1 is one of the motor driver's logic control inputs. The ESP32 uses GPIO 21 to control this signal.

### AIN2

AIN2 is the second motor-driver logic control input. The ESP32 uses GPIO 22 to control this signal.

For this implementation, AIN2 remains LOW while AIN1 changes according to the button state.

### nSLEEP

The nSLEEP input controls whether the motor driver is enabled.

During startup, the program first drives nSLEEP LOW:

```cpp
digitalWrite(DRIVER_SLEEP_PIN, LOW);
```

After the other control pins have been initialized, nSLEEP is driven HIGH:

```cpp
digitalWrite(DRIVER_SLEEP_PIN, HIGH);
```

A short delay is then provided for driver startup.

### VM

VM is the motor supply input of the driver. It supplies the power required by the motor side of the driver and should be connected according to the motor-driver datasheet and the rated voltage of the actual motor.

The ESP32 should not be used as the motor's power source unless the specific hardware setup explicitly supports it.

### Common Ground

The ESP32 ground and the motor-driver logic ground must share a common reference so that the driver's logic inputs can correctly interpret the ESP32 control signals.

The motor power supply should also be connected according to the motor-driver breakout's documented grounding requirements.

---

## 7. Source Code

The main application is located at:

```text
src/main.cpp
```

The implemented program initializes the motor-driver control pins, configures the push button with an internal pull-up, enables the driver, and continuously updates AIN1 according to the button state.

```cpp
#include <Arduino.h>

// Example 7: Button-Controlled Motor Run/Coast

const uint8_t BUTTON_PIN = 23;
const uint8_t MOTOR_IN1 = 21;
const uint8_t MOTOR_IN2 = 22;
const uint8_t DRIVER_SLEEP_PIN = 27;

void setup() {
  pinMode(DRIVER_SLEEP_PIN, OUTPUT);
  digitalWrite(DRIVER_SLEEP_PIN, LOW);

  pinMode(MOTOR_IN1, OUTPUT);
  pinMode(MOTOR_IN2, OUTPUT);

  digitalWrite(MOTOR_IN1, LOW);
  digitalWrite(MOTOR_IN2, LOW);

  pinMode(BUTTON_PIN, INPUT_PULLUP);

  digitalWrite(DRIVER_SLEEP_PIN, HIGH);

  delay(2);
}

void loop() {
  const bool buttonPressed = (digitalRead(BUTTON_PIN) == LOW);

  digitalWrite(MOTOR_IN2, LOW);
  digitalWrite(MOTOR_IN1, buttonPressed ? HIGH : LOW);
}
```

---

## 8. Program Operation

The program follows this sequence:

1. Configure nSLEEP as an output.
2. Keep the motor driver disabled during initialization.
3. Configure AIN1 and AIN2 as outputs.
4. Set both motor-control inputs LOW.
5. Configure the push button using `INPUT_PULLUP`.
6. Enable the motor driver.
7. Continuously read the button.
8. Set AIN1 HIGH when the button is pressed.
9. Set AIN1 LOW when the button is released.
10. Keep AIN2 LOW throughout the basic control program.

This creates a simple direct relationship between the button input and the motor-driver control signal.

---

## 9. PlatformIO Configuration

The project uses the ESP32 Dev Module with the Arduino framework.

The `platformio.ini` configuration is:

```ini
[env:esp32dev]
platform = espressif32
board = esp32dev
framework = arduino
monitor_speed = 115200
```

The project was successfully compiled using PlatformIO.

---

## 10. Project Structure

```text
Laboratory-Activity-6-Basic-Actuator-Control/
│
├── .gitignore
├── Laboratory_6.ino
├── README.md
├── platformio.ini
│
├── include/
│
├── lib/
│
├── src/
│   └── main.cpp
│
└── test/
```

The PlatformIO-generated `.pio/` directory is a local build directory and is not included as project source code.

---

## 11. Build Verification

The project was verified using:

```text
pio run
```

The build completed successfully.

PlatformIO reported:

```text
=== [SUCCESS] ===
```

The successful build confirms that the ESP32 Arduino project configuration and source code compile correctly for the selected ESP32 Dev Module environment.

---

## 12. Optional PWM Extension

The basic implementation can be extended by replacing the simple HIGH/LOW AIN1 control with PWM.

PWM would allow the motor-driver input to receive a variable duty cycle rather than only a fixed HIGH or LOW signal.

The optional extension can be used to investigate:

* Motor-speed control
* PWM duty cycle
* Motor response at different duty cycles
* The duty cycle at which the actual motor begins rotating

The onset-of-rotation value should be determined experimentally using the actual motor and power supply rather than assuming a fixed value.

---

## 13. Hardware and Datasheet Documentation

The final hardware documentation should use the datasheets corresponding to the actual motor and motor-driver breakout used in the laboratory.

Important specifications to document include:

* Motor rated voltage
* Motor stall current
* Motor-driver supply-voltage limits
* Motor-driver output-current limits
* Logic-voltage requirements
* Relevant control-pin functions

These specifications must be taken directly from the actual hardware documentation.

---

## 14. Safety and Power Considerations

Motor loads can draw substantially more current than an ESP32 GPIO pin can provide. The ESP32 GPIO pins are therefore used only as logic-control signals for the motor driver.

The motor should receive power through the appropriate motor-driver power path and external supply specified for the actual hardware.

Before powering the circuit, verify:

* Motor-driver power connections
* ESP32 ground connection
* Common ground between the ESP32 and driver
* Motor connections
* Logic-input connections
* Motor supply voltage
* Current capability of the power supply

---

## 15. GitHub Version Control

The project is maintained using Git and GitHub.

Development progress is committed incrementally so that each significant stage of the laboratory project is recorded separately.

The initial PlatformIO project and motor-control implementation were successfully committed and pushed to the repository.

---

## 16. Documentation Video

The completed documentation video can be added to the project repository and linked here:

[Watch the Documentation Video](./documentation.mp4)

---

## 17. Conclusion

This laboratory activity implements a basic ESP32 actuator-control system using a motor-driver interface and a push button.

The project demonstrates GPIO input handling, digital output control, motor-driver enable control through nSLEEP, and the relationship between AIN1/AIN2 logic inputs and motor operation.

The project is organized using PlatformIO and has been successfully compiled for the ESP32 Dev Module. The optional PWM extension provides a foundation for further investigation of motor-speed control and experimentally determining motor response.
