---
layout: home
permalink: index.html

# Please update this with your repository name and project title
repository-name: eYY-3yp-Syncrobots
title: Infrastructure-Free UWB Swarm Formation Control
---

[comment]: # "This is the standard layout for the project, but you can clean this and use your own template"

# Project Title

---

## Team
-  e22044, Nithika Bandara, [email](mailto:e22044@eng.pdn.ac.lk)
-  e22135, Thimira Dilshan, [email](mailto:e22135@eng.pdn.ac.lk)
-  e22353, Gayasha Sandeepa, [email](mailto:e22353@eng.pdn.ac.lk)
-  e22421, Tharindu Weerasinghe, [email](mailto:e22421@eng.pdn.ac.lk)

<!-- Image (photo/drawing of the final hardware) should be here -->

<!-- This is a sample image, to show how to add images to your page. To learn more options, please refer [this](https://projects.ce.pdn.ac.lk/docs/faq/how-to-add-an-image/) -->

<!-- ![Sample Image](./images/sample.png) -->

#### Table of Contents
1. [Introduction](#introduction)
2. [Solution Architecture](#solution-architecture )
3. [Hardware & Software Designs](#hardware-and-software-designs)
4. [Testing](#testing)
5. [Detailed budget](#detailed-budget)
6. [Conclusion](#conclusion)
7. [Links](#links)

## Introduction

Coordinated multi-robot formations are needed in warehouse logistics, precision agriculture, environmental monitoring, and search-and-rescue. Existing demonstrations usually depend on motion-capture systems such as Vicon or OptiTrack, or on GPS. These are expensive, limited to controlled spaces, or unavailable indoors, so swarm robotics research stays out of reach for student groups and small organizations.

This project builds a low-cost swarm that measures its own relative positions and holds a commanded formation in an ordinary indoor space, with no external infrastructure. Each robot carries an Ultra-Wideband (UWB) ranging module that gives centimeter-level distance measurements without cameras, markers, or GPS.

A simple leader-following chain suffers from string instability, where oscillations grow along the chain. To reduce this, each follower tracks its distance to both the global leader and its immediate predecessor. The result is a decentralized, infrastructure-free formation controller built from inexpensive hardware, so more groups can experiment with swarm robotics.

## Solution Architecture

**Topology.** One leader (`R0`) and several followers (`R1`, `R2`, ...). Each follower `Ri` holds two distance constraints: one to `R0` and one to `R(i-1)`.

**On-board data flow (every 20 Hz control cycle):**

1. UWB SDS-TWR ranging produces raw distances to the leader and the predecessor.
2. A 1D Extended Kalman Filter suppresses the 5 to 15 cm noise and multipath spikes.
3. A potential-field controller combines the filtered distances with the heading estimate from the IMU and wheel encoders.
4. A unicycle mapping converts the controller output into left and right wheel commands, which drive the motors by PWM.

**Communication.**
- **Ranging:** Symmetric Double-Sided Two-Way Ranging (SDS-TWR), which cancels clock drift without synchronized clocks.
- **Scheduling:** a 50 ms TDMA frame gives each robot a 10 ms slot, so the whole swarm refreshes at 20 Hz.
- **Telemetry:** the leader sends periodic summaries over ESP-NOW to a base-station laptop, which logs distances, control errors, and battery voltage to CSV.

**Decentralized control.** Every follower computes its own steering, so the swarm keeps running if the base station goes offline. Only the desired formation distances are set globally.

**Security.** ESP-NOW is unauthenticated by default, so each robot keeps a whitelist of paired MAC addresses and discards packets outside its TDMA slot. No personal data is collected, only robot telemetry.

## Hardware and Software Designs

### Hardware

All nodes use the same bill of materials, so any unit can act as the leader or a follower.

| Function | Component |
|---|---|
| Microcontroller | ESP32 NodeMCU (30-pin) |
| UWB ranging | Decawave/Qorvo DWM1000 / DW3000 module (SPI) |
| Heading | MPU6050 6-DoF IMU and N20 magnetic encoders |
| Drive | 2 x N20 6 V gear motors, MX1508 dual motor driver |
| Chassis | 2-wheel round double-deck smart car chassis, 65 mm wheels |
| Power | 3S 18650 pack, 3S BMS, LM2596 buck (6 V motor rail), Mini-360 buck (5 V logic rail) |
| Level shifting | 4-channel 3.3 V to 5 V logic level converter |

#### Power architecture

The battery pack feeds the BMS, then a rocker switch, then two buck converters. The LM2596 supplies the 6 V motor rail to the MX1508 driver. The Mini-360 supplies 5 V to the ESP32, whose onboard regulator provides 3.3 V for the UWB module and IMU, since neither is 5 V tolerant. All grounds are common.

### Software

- **Firmware (ESP32):** UWB driver and SDS-TWR exchange, TDMA scheduler, Kalman filter, potential-field controller, unicycle mapping, motor PWM, and ESP-NOW with MAC whitelisting.
- **Antenna delay calibration:** each UWB module is calibrated against a known distance before use.
- **Base station:** a script that receives telemetry, writes CSV logs, and plots distance-tracking error and formation shape.

## Testing

| Test | What it checks |
|---|---|
| UWB ranging accuracy | Measured vs. true distance at several known distances, before and after antenna delay calibration |
| Range noise and filtering | Raw vs. Kalman-filtered distance noise on a stationary pair |
| TDMA timing | Whether every robot gets its ranging slot at 20 Hz with no collisions |
| Heading estimation | Yaw drift from gyro and encoder dead reckoning over a run |
| Single-follower tracking | Leader moving at up to 0.3 m/s, tracking error of one follower |
| Full formation | Leader plus two followers holding the commanded formation |
| Battery life | Run time of continuous formation driving per charge |
| Link loss | Whether followers keep operating with the base station switched off |

## Detailed budget

All items and costs

| Item          | Quantity  | Unit Cost  | Total  |
| ------------- |:---------:|:----------:|-------:|
| Sample item   | 5         | 10 LKR     | 50 LKR |

## Conclusion

What was achieved, future developments, commercialization plans

## Links

- [Project Repository](https://github.com/cepdnaclk/{{ page.repository-name }}){:target="_blank"}
- [Project Page](https://cepdnaclk.github.io/{{ page.repository-name}}){:target="_blank"}
- [Department of Computer Engineering](http://www.ce.pdn.ac.lk/)
- [University of Peradeniya](https://eng.pdn.ac.lk/)

[//]: # (Please refer this to learn more about Markdown syntax)
[//]: # (https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet)
