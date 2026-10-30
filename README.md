# ELEGOO Smart Robot Car Kit V4.0 New

[Chinese version](./README.zh-CN.md)

ELEGOO Smart Robot Car Kit V4.0 New is an Arduino-based robot car project for the ELEGOO Smart Robot Car Kit V4.0 platform. The current firmware version is `2.1.2`, and the main Arduino sketch is `ELEGOO-Smart-Robot-Car-Kit-V4.0-New.ino` (the sketch folder must keep the same name for the Arduino toolchain to build it).

This repository contains the core firmware for motion control, line tracking, obstacle avoidance, object following, IR remote control, serial JSON control, RGB lighting, servo control, ultrasonic ranging, voltage detection, and key input handling.

## Features

- Basic movement control: forward, backward, left, right, and stop
- Autonomous modes: line tracking, obstacle avoidance, and object following
- Remote control modes: IR remote and serial rocker control
- Sensor support: line tracking sensors, ultrasonic distance sensor, voltage detection, and MPU6050
- Peripheral control: RGB LED, dual-axis servo, and motor driver
- Serial communication: JSON commands over the serial port
- Watchdog support: AVR watchdog is enabled for better runtime stability

## Project Structure

```text
.
|-- ELEGOO-Smart-Robot-Car-Kit-V4.0-New.ino  # Arduino main sketch
|-- ApplicationFunctionSet.*     # Application state machine, serial commands, mode switching
|-- MotionControl.*              # Motion control, tracking, obstacle avoidance, follow logic
|-- DeviceDriverSet.*            # Motor, servo, IR, ultrasonic, MPU6050, and low-level drivers
|-- RgbControl.*                 # RGB LED control
|-- Utils.*                      # Utility functions
`-- ArduinoJson-v6.11.1.h        # Bundled ArduinoJson header
```

## Community Hardware and Licensing

The chassis in the kit is the stock acrylic plate set. Custom 3D-printed
chassis designs built around the kit's own hardware (motors, wheels, UNO and
shield, servo head, sensors, battery pack) are maintained in the community
fork at https://github.com/westondunn/elegoo-smart-car-v4 (pull requests [#1](https://github.com/westondunn/elegoo-smart-car-v4/pull/1) and [#2](https://github.com/westondunn/elegoo-smart-car-v4/pull/2)).
They do not change the firmware.

This repository ships without a license file, so the firmware remains under
ELEGOO's terms. Material from the official ELEGOO repositories (schematics,
datasheets, chassis plate models) is referenced by pinned commit rather than
copied, and the community fork assigns no license to it.

## Hardware Pin Map

The default pin definitions are already configured in the source code and usually do not need to be changed.

| Module | Pin |
| --- | --- |
| RGB LED | D4 |
| Button | D2 |
| IR receiver | D9 |
| Left line tracking sensor | A2 |
| Middle line tracking sensor | A1 |
| Right line tracking sensor | A0 |
| Voltage detection | A3 |
| Right motor PWM | D5 |
| Left motor PWM | D6 |
| Right motor direction | D7 |
| Left motor direction | D8 |
| Motor STBY | D3 |
| Ultrasonic TRIG | D13 |
| Ultrasonic ECHO | D12 |
| Z-axis servo | D10 |
| Y-axis servo | D11 |
| MPU6050 | I2C SDA/SCL |

## Development Environment

Recommended tools:

- Arduino IDE 2.x
- Or Visual Studio Code with the Arduino extension
- Board: `Arduino Uno`
- Serial baud rate: `9600`

Required Arduino libraries:

- `FastLED`
- `Servo`
- `IRremote`
- `MPU6050`
- `Wire`, included with Arduino

The project includes `ArduinoJson-v6.11.1.h`, so installing ArduinoJson separately is usually not required.

## Build and Upload

### Arduino IDE

1. Open `ELEGOO-Smart-Robot-Car-Kit-V4.0-New.ino`.
2. Select `Tools > Board > Arduino AVR Boards > Arduino Uno`.
3. Connect the robot car controller board by USB.
4. Select the correct serial port, such as `COM3`.
5. Click `Verify` to compile.
6. Click `Upload` to flash the firmware.

After uploading, open Serial Monitor and set the baud rate to `9600`. After reset, the board should print:

```text
Smart Robot Car V4.0 - System Ready
```

### arduino-cli

Compile:

```bash
arduino-cli compile --fqbn arduino:avr:uno .
```

Upload, replacing `COM3` with the actual serial port:

```bash
arduino-cli upload -p COM3 --fqbn arduino:avr:uno .
```

## Serial JSON Control

The firmware receives JSON commands over the serial port. Each message must start with `{` and end with `}`. The maximum message length is `128` bytes.

Common fields:

| Field | Description |
| --- | --- |
| `N` | Command number |
| `H` | Command serial number for response tracking |
| `D1`, `D2`, `D3`, `D4` | Command parameters |
| `T` | Timer duration in milliseconds |

Example: move forward at speed `120` for `1000` ms, then stop. `N=2` is the timed variant; `N=3` runs until the next command and ignores `T`.

```json
{"N":2,"H":"1","D1":3,"D2":120,"T":1000}
```

Example: switch to line tracking mode.

```json
{"N":101,"H":"2","D1":1}
```

Example: set the RGB LED to red for `1000` ms. `N=7` is the timed variant; `N=8` is untimed and ignores `T`.

```json
{"N":7,"H":"3","D1":1,"D2":255,"D3":0,"D4":0,"T":1000}
```

## Main Commands

| `N` | Function |
| --- | --- |
| `1` | Single or all-motor control |
| `2` / `3` | Car direction and speed control (`2` is timed by `T`, `3` is untimed) |
| `4` | Independent left/right motor speed control |
| `5` | Servo angle control |
| `7` / `8` | RGB lighting control (`7` is timed by `T`, `8` is untimed) |
| `21` | Read ultrasonic status or distance |
| `22` | Read line tracking sensor value |
| `23` | Read ground detection status |
| `100` | Clear functions and enter standby |
| `101` | Mode switch: `1` tracking, `2` obstacle avoidance, `3` following |
| `102` | Rocker control mode |
| `105` | LED brightness adjustment |
| `106` | Servo direction control |
| `110` | Clear functions and enter idle state |

## Operating Modes

| Mode | Description |
| --- | --- |
| `Standby` | Standby mode |
| `TraceBased` | Line tracking mode |
| `ObstacleAvoidance` | Obstacle avoidance mode |
| `Follow` | Object following mode |
| `Rocker` | Rocker or remote control mode |
| `DoNothing` | Idle state |

## Troubleshooting

### `FastLED.h` was not found

Open Library Manager in Arduino IDE, search for `FastLED`, and install it.

### `IRremote.hpp` or `MPU6050.h` was not found

Install the corresponding library and compile again. Library APIs may differ between versions, so prefer the library versions from the ELEGOO official package when available.

### Upload failed

Check the following:

- Board is set to `Arduino Uno`
- Correct serial port is selected
- USB cable supports data transfer
- Serial Monitor or another program is not using the COM port
- Board driver is installed correctly

### The car does not move

Check the battery level, power switch, motor wiring, and motor driver STBY control. The firmware uses a battery voltage threshold of `7.00V`; low voltage may cause abnormal behavior.

## Version

- Firmware version: `2.1.2`
- Main sketch: `ELEGOO-Smart-Robot-Car-Kit-V4.0-New.ino`
- Author: ELEGOO
