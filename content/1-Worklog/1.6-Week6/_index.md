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

| Workstream | Work completed | Result/Evidence |
| :--- | :--- | :--- |
| Wiring and power safety | Reviewed GPIO definitions in `hardware/src/main.cpp`, connected the DHT20, light sensor, fan, light/relay, servo, and LCD1602, and checked common ground/power | The verified wiring table followed the firmware; the fan and servo were not powered directly from GPIO |
| Firmware environment | Configured PlatformIO, the ESP32-S3 board definition, and local `secrets.h` | Kept Wi-Fi credentials and the backend URL out of Git |
| Sensors and display | Read DHT20 and raw analog light values and updated the LCD1602 | Serial Monitor/LCD displayed sensor data and state |
| Actuator control | Tested the fan, light/relay, and servo at 0° closed/90° open | Actuators responded according to firmware logic |
| REST API integration | Sent periodic telemetry, polled commands, executed them, and returned ACK | YOLO UNO communicated bidirectionally with FastAPI over HTTP |
| Build and evidence | Built the firmware, reviewed Serial Monitor, and recorded the demo | PlatformIO generated `firmware.bin`; video captured physical operation |
## Weekly outcomes

- Sent `room_01` telemetry and received backend commands on YOLO UNO.
- Controlled the fan, light, and curtain servo through direct commands; Auto mode used firmware-defined thresholds.
- Used `lastAck` and `pendingAck` state to retry acknowledgement without repeating a physical action.

## Challenges and lessons learned

Safe actuator power and duplicate-execution protection were as important as network connectivity. After an ACK failure, the firmware must retry only the ACK and not operate the actuator again.

## Workshop references

- [5.6 Hardware Integration](../../5-workshop/5.6-hardware-integration/)
- [5.8 End-to-End Testing](../../5-workshop/5.8-end-to-end-testing/)
