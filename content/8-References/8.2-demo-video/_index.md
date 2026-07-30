---
title: "Demo Video"
date: "2026-07-30"
weight: 2
chapter: false
pre: " <b> 8.2. </b> "
---

The video demonstrates the end-to-end operation of the **AWS IoT Monitoring and Control Dashboard**.

## Demo coverage

The video demonstrates:

1. The React + Vite dashboard receiving near-real-time telemetry.
2. The FastAPI backend running on Amazon EC2.
3. Telemetry and commands stored in Amazon RDS for PostgreSQL.
4. YOLO UNO polling commands from the backend.
5. The fan, light, and curtain servo responding to control commands.
6. The device sending an ACK after execution.
7. The backend updating the command state to `Executed`.

The video does not include the Amazon CloudWatch console. Evidence for logs, metrics, and alarms is provided through screenshots in [section 5.9 - Monitoring with CloudWatch]({{% relref "5-Workshop/5.9-CloudWatch-Monitoring/_index.md" %}}).

## Video link

[▶ Watch the end-to-end demo](https://drive.google.com/file/d/1T97dUY58hbT2ppxvg7ESR12Jg9BA828W/view?usp=sharing)
