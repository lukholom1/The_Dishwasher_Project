# The_Dishwasher_Project
Dishwasher logic board Replaced with ESP32
# Dishwasher Retrofit Control Board

A custom ESP32-S2 based controller that replaces the original control board of a standard dishwasher. Built using off-the-shelf modules wired to an ESP32-S2 DevKit, this project brings full programmable cycle control, real-time temperature monitoring, and a live OLED display to any dishwasher — old or new.

---

## Overview

The Dishwasher Retrofit Control Board takes over the core functions of a dishwasher's original PCB. It controls the water inlet solenoid, drain pump, wash motor, and heating element through a 4-channel relay module, and monitors water temperature via a DS18B20 sensor. A 128x64 OLED display shows live cycle status, phase progress, time remaining, and I/O state. A physical start/stop button gives the user full control, and a dedicated safety pin provides a hardware-level emergency cut-off.

---

## Hardware

| Component | Role |
|---|---|
| ESP32-S2 DevKitM-1 | Main microcontroller |
| 4-Channel Relay Module | Switches mains-voltage dishwasher components |
| DS18B20 Temperature Sensor | Monitors wash water temperature |
| SSD1306 OLED (128x64) | Live cycle display |
| Pushbutton (pin 7) | Start / Pause / Resume / Reset |
| Slide Switch (pin 5) | Simulated water level sensor |
| Safety Wire (pin 8) | Hardware emergency cut-off |

---

## Pin Assignments

| Pin | Component |
|---|---|
| GPIO 1 | Relay 1 — Water Inlet Solenoid |
| GPIO 3 | Relay 2 — Drain Pump |
| GPIO 20 | Relay 3 — Wash Motor |
| GPIO 18 | Relay 4 — Heating Element |
| GPIO 46 | DS18B20 Temperature Sensor (DQ) |
| GPIO 35 | OLED SDA |
| GPIO 36 | OLED SCL |
| GPIO 37 | OLED VCC (software powered) |
| GPIO 5 | Water Level Switch |
| GPIO 7 | Start / Stop Button |
| GPIO 8 | Safety Cut-Off |

---

## Wash Cycle

The controller runs a 6-phase wash cycle that mirrors a full 60-minute dishwasher program. For development and testing, the full cycle is simulated in 2 minutes (1 simulated second = 30 real seconds).

| Phase | Real Time | Sim Time | Solenoid | Motor | Heater | Drain | Target Temp |
|---|---|---|---|---|---|---|---|
| Pre-Wash | 5 min | 10 s | ✅ | ✅ | ❌ | ❌ | — |
| Main Wash | 25 min | 50 s | ✅ | ✅ | ✅ | ❌ | 55°C |
| Rinse 1 | 5 min | 10 s | ✅ | ✅ | ❌ | ❌ | — |
| Hot Rinse | 15 min | 30 s | ✅ | ✅ | ✅ | ❌ | 90°C |
| Drain | 5 min | 10 s | ❌ | ❌ | ❌ | ✅ | — |
| Dry | 5 min | 10 s | ❌ | ❌ | ❌ | ❌ | Passive |

### Drying Method — Residual Heat Drying
No active heater or fan is used during the dry phase. The Hot Rinse cycle heats both the water and the dishes to 90°C. Once the water drains, the thermal energy stored in the dishes causes any remaining water droplets to evaporate naturally. This is energy efficient and requires no additional hardware.

---

## Controls

### Start / Stop Button (pin 7)
Wire one leg to GPIO 7 and the other directly to GND. No resistor needed — the ESP32 internal pull-up is used.

| Machine State | Button Press | Result |
|---|---|---|
| Idle | Press | Starts cycle |
| Running | Press | Pauses cycle, all relays off |
| Paused | Press | Resumes from exact point |
| Complete | Press | Resets, ready for new cycle |

Pause is non-destructive — the controller tracks how many seconds were spent in the current phase before pausing and resumes from that point exactly.

### Safety Cut-Off (pin 8)
Wire pin 8 to GND through whatever safety mechanism is appropriate for your installation — a door interlock microswitch, a float switch, or a simple jumper wire for testing.

- **Pin 8 LOW (GND connected)** = safe, cycle runs normally
- **Pin 8 HIGH (connection lost)** = immediate emergency stop

On a safety trigger, all four relays cut off instantly, the OLED displays a `!! SAFETY STOP !!` screen, and the controller blocks until GND is restored. Once the connection is re-established, the cycle resumes automatically.

---

## OLED Display

The display updates in real time throughout the cycle and shows:

- Current phase name and phase number (e.g. `Phase: HOT RINSE  4/6`)
- Full-cycle progress bar
- Time remaining in the current phase, converted to real minutes and seconds
- Live water temperature in °C
- Active relay status (`SOL`, `DRN`, `MTR`, `HTR`)
- Special labels for paused (`!! PAUSED !!`), passive dry (`** PASSIVE DRY **`), and safety stop states

---

## Libraries Required

Install these via the Arduino Library Manager or Wokwi's library panel:

- Adafruit SSD1306
- Adafruit GFX Library
- OneWire
- DallasTemperature

---

## Simulation (Wokwi)

The project was developed and tested in [Wokwi](https://wokwi.com), an online ESP32 simulator. The water level switch in simulation is a slide switch on pin 5 — flip it to HIGH to simulate the tank reaching full level during fill phases. The cycle will not advance past a fill phase until the level switch is triggered.

To simulate the full 60-minute cycle at real speed, change the `SCALE_FACTOR` constant from `30.0` to `1.0`.

---

## Author

Lukholo — Dishwasher Retrofit Control Board  
Built as part of a hands-on embedded systems project.
