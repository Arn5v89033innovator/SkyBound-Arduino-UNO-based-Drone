# YMFC-AL — Your Multirotor Flight Controller (Auto-Level)

An open-source Arduino-based quadcopter flight controller with auto-leveling, built around the MPU-6050 IMU. YMFC-AL uses a complementary filter to fuse gyroscope and accelerometer data for stable, self-leveling flight with PID control on roll, pitch, and yaw.

> ⚠️ **Safety Warning:** Always remove propellers and keep a safe distance from motors unless you are 100% certain of what you are doing.

---

## Features

- **Auto-level mode** — complementary filter merges gyro + accelerometer data for hands-off stability
- **PID controller** — independent tunable P, I, D gains for roll, pitch, and yaw
- **MPU-6050 support** — I²C communication at 400 kHz for fast sensor reads
- **4-channel RC receiver input** — standard PWM signals (1000–2000 µs) via pin-change interrupts
- **4 ESC outputs** — direct PWM signals to brushless motor ESCs
- **EEPROM configuration** — stores gyro type, axis mapping, and calibration data from the setup sketch
- **Battery voltage monitoring** — analog input with compensation for voltage drop
- **Guided setup wizard** — serial-based setup sketch handles receiver mapping, gyro detection, and ESC calibration

---

## Hardware Requirements

| Component | Details |
|-----------|---------|
| Microcontroller | Arduino Uno / Nano (ATmega328P, 16 MHz) |
| IMU | MPU-6050 (gyroscope + accelerometer) |
| RC Receiver | 4-channel PWM receiver |
| ESCs | 4× standard PWM ESCs |
| Battery | 3S LiPo (voltage monitored via Analog 0) |

### Pin Mapping

| Arduino Pin | Function |
|-------------|----------|
| Digital 4 | ESC 1 output |
| Digital 5 | ESC 2 output |
| Digital 6 | ESC 3 output |
| Digital 7 | ESC 4 output |
| Digital 8 | RC Channel 1 (Roll) |
| Digital 9 | RC Channel 2 (Pitch) |
| Digital 10 | RC Channel 3 (Throttle) |
| Digital 11 | RC Channel 4 (Yaw) |
| Digital 12 | Status LED |
| Analog 0 | Battery voltage |
| SDA / SCL | MPU-6050 (I²C) |

See `YMFC_schematic.pdf` for the full wiring diagram.

---

## Repository Contents

```
YMFC-AL/
├── YMFC-AL_setup/
│   └── YMFC-AL_setup.ino         # Step-by-step setup wizard (run first)
├── YMFC-AL_esc_calibrate/
│   └── YMFC-AL_esc_calibrate.ino # ESC throttle range calibration
├── YMFC-AL_Flight_controller/
│   └── YMFC-AL_Flight_controller.ino  # Main flight controller firmware
├── YMFC_schematic.pdf            # Full wiring schematic
└── ReadMe.txt                    # Original release notes
```

---

## Getting Started

### Step 1 — Wire the hardware

Follow `YMFC_schematic.pdf` to connect the MPU-6050, receiver, ESCs, and battery.

### Step 2 — Calibrate ESCs

Open `YMFC-AL_esc_calibrate/YMFC-AL_esc_calibrate.ino` in the Arduino IDE and upload it. Follow the serial monitor prompts to set the throttle range on all four ESCs.

### Step 3 — Run the setup wizard

Open `YMFC-AL_setup/YMFC-AL_setup.ino` and upload it. Open the Serial Monitor at **57600 baud**. The wizard will:

1. Verify I²C clock speed (must be 16 MHz Arduino for 400 kHz I²C)
2. Detect and configure the RC receiver channel mapping
3. Detect the MPU-6050 and determine gyro axis orientation
4. Store all configuration to EEPROM

You **must** complete this step before uploading the flight controller — the firmware reads its configuration from EEPROM on startup and will halt if the signature is missing.

### Step 4 — Upload the flight controller

Open `YMFC-AL_Flight_controller/YMFC-AL_Flight_controller.ino` and upload it. On power-up, the controller will:

1. Read EEPROM configuration
2. Initialise the MPU-6050
3. Pulse ESCs for 5 seconds while waiting
4. Collect 2000 gyro samples to calculate calibration offsets
5. Wait for a valid receiver signal with throttle low and yaw right (arm sequence)

### Step 5 — Arm and fly

With throttle at minimum and yaw stick fully right, hold for ~2 seconds to arm. The status LED will go solid. Raise throttle to fly.

---

## PID Tuning

Default gains are set in `YMFC-AL_Flight_controller.ino`:

```cpp
float pid_p_gain_roll  = 1.3;
float pid_i_gain_roll  = 0.04;
float pid_d_gain_roll  = 18.0;
int   pid_max_roll     = 400;

float pid_p_gain_yaw   = 4.0;
float pid_i_gain_yaw   = 0.02;
float pid_d_gain_yaw   = 0.0;
int   pid_max_yaw      = 400;
```

Pitch gains mirror roll by default. Adjust these values to suit your frame and motor/prop combination. A good starting approach:

1. Set I and D to zero, increase P until the quad oscillates, then back off.
2. Add D to dampen oscillations.
3. Add a small I to correct steady-state drift.

---

## Auto-Level

Auto-level is enabled by default:

```cpp
boolean auto_level = true;   // Set to false to disable
```

When enabled, the complementary filter blends gyro integration with accelerometer angle estimates to continuously correct roll and pitch towards level. The filter coefficient is 0.9999 (gyro) / 0.0001 (accelerometer) per loop iteration.

---

## Libraries Required

These are bundled with the Arduino IDE — no additional installation needed:

- `Wire.h` — I²C communication with MPU-6050
- `EEPROM.h` — Persistent storage of configuration data

---

## Changelog

### v1.4 — April 30, 2017
Fixed ESC calibration sketch stopping unexpectedly when some Arduino IDE versions send a line feed character alongside serial input. The serial buffer is now flushed after reading.

### v1.3 — October 30, 2016
Fixed setup sketch hang caused by compiler optimisation in Arduino IDE 1.6.x and later. Reduced motor starting speed to 1100 µs. Added PDF schematic.

### v1.2 — July 12, 2016
Completed fix for NaN issue: `abs()` guard added to both `acc_x` and `acc_y` before `asin()` to handle negative values correctly.

### v1.1 — July 11, 2016
Partial fix for NaN (not-a-number) error in accelerometer angle calculation triggered during aggressive manoeuvres or fast descents, where `acc_x` / `acc_y` could exceed `acc_total_vector`.

### v1.0 — July 3, 2016
Initial release.

---

## License

This software is provided **"as is"**, without warranty of any kind, express or implied, including but not limited to the warranties of merchantability, fitness for a particular purpose, and non-infringement. The authors are not liable for any claim, damages, or other liability arising from use of this software.

---

## Resources

- Original project and build tutorial: [brokking.net](http://www.brokking.net/ymfc-al_main.html)
- YouTube build series: search **YMFC-AL** on YouTube for full assembly and tuning guides
