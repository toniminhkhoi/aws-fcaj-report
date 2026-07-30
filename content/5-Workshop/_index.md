---
title: "Workshop"
date: "2026-07-28"
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

This hands-on workshop guides learners through building an AWS-connected Smart Room identified by `device_id=room_01`. YOLO UNO collects temperature, humidity, and analog light readings; the FastAPI backend on Amazon EC2 processes requests and stores telemetry and command states in Amazon RDS for PostgreSQL; and the React dashboard displays data and sends device-control commands. After executing a command, the firmware sends an ACK so the backend can record its final state.

## Objectives and Expected Outcomes

By the end of the workshop, you will be able to:

- prepare the VPC, Security Groups, Amazon EC2, Amazon EBS, and Amazon RDS for PostgreSQL resources used by the project;
- deploy FastAPI on EC2 as a `systemd` service and verify its database connection;
- build and upload the PlatformIO firmware for YOLO UNO;
- collect telemetry and control the fan, light, and curtain through 8 supported firmware commands;
- use the React + Vite dashboard to view current data, review history, and send commands;
- validate the command lifecycle from `Pending` to `Executed` through device ACK;
- inspect backend logs, EC2/RDS metrics, and alarm states in Amazon CloudWatch; and
- complete end-to-end testing, troubleshooting, resource clean-up, and project handover.

The final outcome is a reproducible Smart Room prototype for `device_id=room_01`, supported by source code, deployment instructions, test records, AWS screenshots, and a hardware demonstration.

## Workshop Contents

1. [5.1 Workshop Overview](5.1-Workshop-overview/)
2. [5.2 Prerequisites](5.2-Prerequisites/)
3. [5.3 Architecture and Service Design](5.3-Architecture-and-Service-Design/)
4. [5.4 AWS Infrastructure Setup](5.4-AWS-Infrastructure-Setup/)
5. [5.5 Backend Deployment and Database Integration](5.5-Backend-and-Database/)
6. [5.6 Hardware Integration](5.6-Hardware-Integration/)
7. [5.7 Frontend Integration](5.7-Frontend-Integration/)
8. [5.8 End-to-End Testing and Validation](5.8-End-to-End-Testing/)
9. [5.9 Monitoring with CloudWatch](5.9-CloudWatch-Monitoring/)
10. [5.10 Cost, Security and Clean-up](5.10-Cost-Security-Cleanup/)
11. [5.11 Results, Challenges and Future Improvements](5.11-Results-Challenges-Future/)
12. [5.12 Project Handover](5.12-Project-Handover/)

## Architecture and Operating Flow

![AWS IoT Monitoring and Control Dashboard architecture](/images/5-Workshop/5.3-architecture/aws-iot-dashboard-architecture.png)

*Figure 5-1. Project architecture showing the local React dashboard and YOLO UNO communicating with the FastAPI backend on Amazon EC2, while Amazon RDS for PostgreSQL stores telemetry and command data and Amazon CloudWatch provides operational monitoring.*

The system operates through four main flows:

1. YOLO UNO reads the sensors and submits telemetry to FastAPI over HTTP.
2. FastAPI validates and stores telemetry in Amazon RDS for PostgreSQL.
3. The dashboard retrieves current and historical data, creates commands, and displays their states.
4. YOLO UNO polls for commands, controls the corresponding actuator, and sends an ACK; CloudWatch collects the related operational logs and metrics.

The AWS environment consists of **Amazon VPC, subnets, Security Groups, Amazon EC2 with an Amazon EBS root volume, Amazon RDS for PostgreSQL, an IAM Role attached to EC2, Amazon CloudWatch, and CloudWatch Alarms**.

Begin with [Workshop Overview](5.1-Workshop-overview/).
