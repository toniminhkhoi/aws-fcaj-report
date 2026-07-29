# AWS IoT Monitoring and Control Dashboard 🚀

An AWS-based system for remote environmental monitoring and IoT device control, built with FastAPI, React, PostgreSQL, YOLO UNO, and AWS services.

> The current scope is a prototype for a single room, `room_01`. It is not a multi-branch Building Management System (BMS) or an enterprise-scale operational platform.

---

## 📋 Project Overview

YOLO UNO collects temperature, humidity, and light readings, then sends telemetry over HTTP to a FastAPI backend running on Amazon EC2. The backend stores telemetry and command states in Amazon RDS for PostgreSQL. The React + Vite dashboard displays the latest readings and historical data, and allows operators to control the fan, light, and curtain.

The device polls for pending commands, executes each command once, and sends an ACK so the backend can change its state from `Pending` to `Executed`. Amazon CloudWatch collects backend logs, monitors EC2 and RDS metrics, and evaluates the configured alarms.

- **Report author:** Phạm Lê Minh Khôi
- **Institution:** Ho Chi Minh City University of Technology (HCMUT) – Faculty of Computer Science and Engineering
- **Workshop:** [View the online report](https://danielleit241.github.io/aws-fcj-report/)

---

## 🏛️ System Architecture

```text
┌──────────────────────────────────────────┐
│        React + Vite Dashboard            │
│             (runs locally)               │
└──────────────────────────────────────────┘
                    │ REST API
                    ▼
┌──────────────────────────────────────────┐
│        FastAPI Backend on EC2            │
│         EBS · IAM Role · systemd         │
└──────────────────────────────────────────┘
          │                         │
          ▼                         ▼
 Amazon RDS for              Amazon CloudWatch
   PostgreSQL                 Logs · Metrics · Alarms
          ▲
          │ Telemetry · Polling · ACK
          ▼
┌──────────────────────────────────────────┐
│       YOLO UNO / Python Simulator        │
└──────────────────────────────────────────┘
```

### Main Components

1. **YOLO UNO / Python Simulator:** Sends telemetry, retrieves pending commands, and returns acknowledgements. YOLO UNO is the primary device, while the simulator supports testing when physical hardware is unavailable.
2. **Amazon EC2 and EBS:** Runs the FastAPI backend as a `systemd` service; EBS provides the EC2 root volume.
3. **Amazon RDS for PostgreSQL:** Stores devices, telemetry history, and command lifecycle data.
4. **React + Vite Dashboard:** Displays the latest readings and historical data, and sends device-control requests.
5. **Amazon CloudWatch:** Collects logs, monitors EC2 and RDS metrics, and evaluates alarms.
6. **Amazon VPC, Security Groups, and IAM Role:** Control network connectivity and monitoring permissions according to the principle of least privilege.

The implemented AWS services are **Amazon EC2, Amazon EBS, Amazon RDS for PostgreSQL, Amazon VPC, Security Groups, AWS IAM Role, Amazon CloudWatch, and CloudWatch Alarms**.

AWS IoT Core, Lambda, API Gateway, DynamoDB, S3, Auto Scaling, and Amazon SQS are not implemented in the current version.

---

## 🔄 Main Workflows

- **Telemetry:** YOLO UNO reads sensor values → sends an HTTP POST request to FastAPI → the backend stores the data in PostgreSQL → the dashboard retrieves the latest and historical readings.
- **Device command:** An operator uses the dashboard → FastAPI creates a command in the `Pending` state → the device polls and executes the command → the device sends an ACK → the backend changes the state to `Executed`.
- **Monitoring:** CloudWatch Agent sends EC2 operating-system logs and metrics; CloudWatch also monitors the default EC2 and RDS metrics.

---

## 👥 Team Members and Responsibilities

| Team member | Responsibilities |
| :--- | :--- |
| **Phạm Lê Minh Khôi** | AWS architecture; VPC, Security Groups, IAM Role, EC2, RDS, and CloudWatch; DevOps; YOLO UNO hardware; sensors, actuators, telemetry, command polling, and ACK mechanism |
| **Ngô Minh Thuận** | FastAPI backend; API endpoints, Pydantic schemas, SQLAlchemy models, PostgreSQL integration, telemetry processing, command lifecycle, and ACK handling |
| **Thượng Đình Hưng** | React + Vite frontend; dashboard UI, telemetry visualization, control features, system integration, error handling, and demonstration video production |
| **Lê Bảo Khánh** | Proposal, blog posts, weekly worklogs, event report, bilingual Workshop, navigation, visual evidence, and documentation quality assurance |

Detailed contributions and individual reflections are provided in [section 5.11](content/5-Workshop/5.11-Results-Challenges-Future/_index.md) and [section 5.12](content/5-Workshop/5.12-Project-Handover/_index.md).

---

## 🧰 Technology Stack

- **Backend:** Python, FastAPI, Uvicorn, SQLAlchemy, and Pydantic.
- **Frontend:** React, Vite, TypeScript, and Tailwind CSS.
- **Database:** PostgreSQL on Amazon RDS.
- **Hardware:** YOLO UNO/ESP32-S3, PlatformIO, DHT20, analog light sensor, LCD1602, fan, light/relay, and servo.
- **AWS:** EC2, EBS, RDS, VPC, Security Groups, IAM Role, CloudWatch, and CloudWatch Alarms.
- **Report:** Hugo and `hugo-theme-learn`.

---

## 📚 Report Contents

- `content/1-Worklog/`: Weekly worklogs.
- `content/2-Proposal/`: Project proposal.
- `content/3-BlogsTranslated/`: Translated blog posts.
- `content/4-EventParticipated/`: Participated events.
- `content/5-Workshop/`: Bilingual AWS IoT Dashboard Workshop.
- `content/6-Self-evaluation/`: Self-evaluation.
- `content/7-Feedback/`: Feedback.

The backend, frontend, and firmware source code are maintained in the `aws-iot-dashboard` application repository. This repository contains the internship report and Workshop.

---

## ▶️ Run the Website with Hugo

```powershell
hugo server
```

Then open `http://localhost:1313/`. Use the language selector to switch between **English** and **Tiếng Việt**.

To generate a static build:

```powershell
hugo --minify
```

---

## 🔐 Security Notes

- Do not commit `.env`, `.pem`, private keys, passwords, or `hardware/include/secrets.h`.
- Allow PostgreSQL connections to RDS only from the EC2 Security Group.
- Restrict SSH access to the administrator's IP address.
- Do not hard-code AWS access keys in source code.
- Use an IAM Role for CloudWatch Agent permissions.

---

## 📄 Copyright

Copyright © 2026 Phạm Lê Minh Khôi – HCMUT. All rights reserved.
