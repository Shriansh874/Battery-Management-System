# Battery Management System


A ** single‑cell Li‑Ion battery‑management and monitoring platform** built around an  
**ATtiny1616** microcontroller, high‑side current sensing, and a 2 A USB‑C charger.  
The hardware, firmware, manufacturing files, and documentation in this repository let you:

---

## Table of Contents
1. [Features](#features)  
2. [Hardware Architecture](#hardware-architecture)  
3. [Bill of Materials](#bill-of-materials)  
4. [Firmware Overview](#firmware-overview)  
5. [Getting Started](#getting-started)  
6. [Flashing & Debugging](#flashing--debugging)  
7. [Serial Dashboard](#serial-dashboard)  
8. [Repository Structure](#repository-structure)  
9. [Contributing](#contributing)  
10.

---

## Features
| Capability | Detail |
|------------|--------|
| **Charging** | IP2312 linear charger — 4.20 V CV, 2 A CC |
| **Load switch** | 25 mΩ P‑MOSFET with push‑button control |
| **Measurements** | INA219 (bus V/shunt V/I), TMP102 (temperature), ADS1015 (4 × AIN) |
| **MCU** | ATtiny1616 @ 20 MHz (megaTinyCore) |
| **Cooling Fan** | PA6 output with current‑based fault cut‑off |
| **User I/O** | Two push buttons, reset, tri‑colour LED bar |
| **Telemetry** | 115 200 baud VT100‑style serial dashboard |
| **Standby** | MCU sleeps in `SLEEP_MODE_STANDBY` when idle |

---

## Hardware Architecture

| Block | ICs | Purpose |
|-------|-----|---------|
| **Power Path** | IP2312 • 10 mΩ shunt • P‑MOSFET | USB‑C input → battery → high‑side switch |
| **Sensor Ring** | INA219 • TMP102 • ADS1015 | High‑side current, board temp, 4× A/D |
| **Regulation** | LD56100DPU33R | 3.3 V, 1 A LDO |
| **Controller** | ATtiny1616‑MNR | Button logic, telemetry, fan control |
| **Connectors** | 34 castellated pads + centre pad | Easy SMT drop‑in onto a carrier PCB |

> **PCB:** 4‑layer FR‑4, 1 oz Cu, stitched ground; impedance‑controlled traces on sensor lines.

---

## Bill of Materials (Top‑Level)

| RefDes | Part Number / Value | Notes |
|--------|--------------------|-------|
| IC2 | ATtiny1616‑MNR | 32‑pin QFN |
| IC3 | LD56100DPU33R | LDO, 3.3 V @ 1 A |
| IC4 | IP2312 | 1‑cell charger |
| IC5 | TMP102AIDRLR | ±0.5 °C |
| IC8 | ADS1015IDGSR | 12‑bit ADC |
| IC9 | INA219AIDCNR | Current/voltage monitor |
| Q5 | DMP3013SFV‑7 | 25 mΩ P‑MOSFET |
| L1 | 4.7 µH | π‑filter |
| *All parts* | See `hardware/BOM.csv` |

---

## Firmware Overview
* **Toolchain:** Arduino IDE / CLI with **megaTinyCore**  
* **Clock:** 20 MHz internal; 32.768 kHz RTC for 1 s periodic tasks  
* **I²C Addresses:** `0x40` (INA219) • `0x49` (TMP102) • `0x48` (ADS1015)  
* **Key ISRs:**  
  * `PORTA_PORT_vect` – debounce & edge detection on buttons  
  * `TCA0_OVF_vect` – 100 ms scheduler for short/long presses  
  * `RTC_PIT_vect` – 1 s telemetry update and serial refresh  
* **Safety Logic:** 3‑sample rolling average current; trips fault LED and disables fan above user‑defined threshold.

---

