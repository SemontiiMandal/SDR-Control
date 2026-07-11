# FLRTRX Subsystem C: Microcontroller and Local Oscillator

This repository contains the hardware schematics, firmware, and control software for Subsystem C of the ECE295 Flexible Radio Transceiver (FLRTRX). Developed by Team C3, this subsystem functions as the central control unit and local oscillator for the radio.

<img width="2048" height="1421" alt="shorty_mainboard" src="https://github.com/user-attachments/assets/71b7a612-b62d-422d-8b48-21d56e2dfd1f" />

<img width="2048" height="1421" alt="image" src="https://github.com/user-attachments/assets/716d3513-0b3e-48ca-9bfb-f32d8ac261b0" />

<img width="2048" height="985" alt="image" src="https://github.com/user-attachments/assets/b15f002e-2b89-41c6-a2a4-0d47cc224bbe" />



---
## Note on Academic Policy

This project was completed as part of ECE295 – Hardware Design and Communication at the University of Toronto.
As per the course policy, source code cannot be publicly posted to prevent plagiarism in future years.

If you would like to learn more about the design and implementation details, please feel free to contact me directly, and I’d be happy to discuss the project!

That being said, the following sections outline the project’s features, technical details, and overall hardware architecture.

---

## Demo and Slides

https://drive.google.com/file/d/1BpMP5pkElbUJRFESBloqsJYpjrlwl7wS/view?usp=sharing

https://drive.google.com/file/d/14NNV1Wk9cgnnpGcA3bdKxI6gW5AK1AMu/view?usp=drive_link

---

## System Overview

The subsystem generates the reference signals required for RF mixing and manages the operating state of the transceiver. It provides a bidirectional control interface, allowing operation via a physical front panel or a remote computer.

The hardware is based on a custom printed circuit board utilizing a Microchip ATmega324PB microcontroller and a Silicon Labs Si5351 clock generator.

## Hardware Specifications

* **Microcontroller:** ATmega324PB
* **Clock Generator:** Si5351 (I2C interface)
* **PC Interface:** FTDI USB-to-UART module
* **Physical Interface:** Character LCD, rotary encoder, hardware TX/RX toggle switch

## Firmware Architecture

The firmware is written in C and manages state synchronization, RF synthesis, and system failsafes.

### Signal Generation

The Si5351 generates two quadrature signals (I and Q) operating between 8 and 16 MHz. To achieve the required 90-degree phase difference without the jitter inherent to integer mode, the firmware configures the clock generator using fractional mode arithmetic. The fractional parameter is minimized (`A + B/C` where the fraction approaches 0) to maintain phase lock while suppressing phase noise.

### State Management and Failsafes

* **Initialization Delay:** The system executes a mandatory 5-second boot sequence to allow the phase-locked loops (PLL) to stabilize before signal generation begins.
* **State Persistence:** The active frequency and operating mode are continuously written to EEPROM. In the event of power loss, the system restores the last known configuration upon reboot.
* **Hardware Protection:** A watchdog timer (WDT) monitors the main execution loop and forces a reset if the microcontroller or connected peripherals hang. A time-out timer automatically disables transmission after three minutes of continuous operation.
* **Input Handling:** Simultaneous commands from the physical interface and the PC console are processed using a last-command, first-execution priority system to prevent state conflicts. Tuning speed adapts to the rotary encoder velocity; fast rotations yield 0.1 MHz steps, while slow rotations yield 0.001 MHz steps.

## Software Control Console

A Python-based application provides remote control and telemetry logging. It interfaces with the hardware via USB-UART.

* **Communication Protocol:** The software uses standard CAT string commands (`TX`, `FA`, `FB`, `IF`).
* **Logging:** Frequency changes and power faults are automatically recorded to a CSV file to assist in debugging silent hardware crashes.

## Interface Control Document (ICD) Validation

The assembled system was validated against the project requirements using an oscilloscope and spectrum analyzer.

* **Frequency Range:** 8 – 16 MHz
* **Frequency Error:** ±650 Hz (Requirement: < ±1000 Hz)
* **Output Voltage:** 3.3 Vpp
* **Phase Difference:** 90° ± 10° (Requirement: 90° ± 12.5°)

## Building and Flashing

**Requirements:**

* AVR Toolchain (`avr-gcc`, `avr-libc`, `avrdude`)
* Python 3.x (with `pyserial`)
* ISP Programmer (e.g., Atmel-ICE)

**Instructions:**

1. Navigate to the `firmware` directory.
2. Compile the source code using the provided makefile: `make all`
3. Flash the ATmega324PB: `make flash`
4. To run the PC console, navigate to the `software` directory and execute: `python gui.py --port <COM_PORT> --baud 115200`
