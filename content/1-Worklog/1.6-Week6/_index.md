---
title: "Week 6 - YOLO UNO Hardware Integration"
date: "2026-07-06"
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

> **Period:** 6–12 July 2026
> **Role:** Led YOLO UNO hardware and firmware integration and coordinated telemetry, command, and ACK testing with the backend member.

## Objectives

- Connect sensors and actuators according to the active firmware.
- Build, upload, and run the firmware with PlatformIO.
- Validate Wi-Fi, telemetry, command polling, and ACK against the backend.

## Work completed

| Period | Activity | Recorded outcome |
| :--- | :--- | :--- |
| 6 July | Reviewed GPIO definitions in `hardware/src/main.cpp` and connected the DHT20, light sensor, fan, light/relay, servo, and LCD1602 | Used the running firmware as the wiring source of truth; the servo used D11/GPIO38 |
| 7 July | Reviewed power delivery, common ground, and safe actuator states | Avoided powering the fan or servo directly from a GPIO pin |
| 8 July | Configured PlatformIO, the ESP32-S3 board definition, and local `secrets.h` | Kept Wi-Fi credentials, the backend URL, and `room_01` out of Git |
| 9 July | Tested the DHT20, raw analog light input, fan, light, 0°/90° servo positions, and LCD | Confirmed sensor and actuator behavior against firmware logic |
| 10–11 July | Sent telemetry every five seconds and polled commands every two seconds before execution and ACK | Connected YOLO UNO to FastAPI over HTTP and handled the eight supported commands |
| 12 July | Built the firmware, reviewed Serial Monitor output, and recorded the demo | Generated `firmware.bin` and captured physical hardware operation |

## Weekly outcomes

- Sent `room_01` telemetry and received backend commands on YOLO UNO.
- Controlled the fan, light, and curtain servo through direct commands; Auto mode used firmware-defined thresholds.
- Used `lastAck` and `pendingAck` state to retry acknowledgement without repeating a physical action.

## Challenges and lessons learned

Safe actuator power and duplicate-execution protection were as important as network connectivity. After an ACK failure, the firmware must retry only the ACK and not operate the actuator again.

## Workshop references

- [5.6 Hardware Integration](../../5-workshop/5.6-hardware-integration/)
- [5.8 End-to-End Testing](../../5-workshop/5.8-end-to-end-testing/)
