---
title: "Workshop"
date: "2026-07-28"
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

This hands-on workshop guides learners through building an AWS-connected Smart Room identified by `device_id=room_01`. CloudFront/WAF serves a private-S3 React dashboard and forwards browser `/api/*` traffic to an ALB. The ALB distributes requests across two FastAPI instances in an ASG; RDS PostgreSQL Multi-AZ stores telemetry and command states. YOLO UNO uses the ALB directly, executes commands, and sends ACK.

## Objectives and Expected Outcomes

By the end of the workshop, you will be able to:

- prepare CloudFront/WAF/private S3, ALB/ASG/EC2/EBS, VPC/Security Groups, and RDS PostgreSQL Multi-AZ resources;
- deploy FastAPI on EC2 as a `systemd` service and verify its database connection;
- build and upload the PlatformIO firmware for YOLO UNO;
- collect telemetry and control the fan, light, and curtain through 8 supported firmware commands;
- use the React + Vite dashboard to view current data, review history, and send commands;
- validate the command lifecycle from `Pending` to `Executed` through device ACK;
- inspect backend logs, ALB/ASG/EC2/RDS metrics, and alarm states in Amazon CloudWatch; and
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

![AWS IoT Monitoring and Control Dashboard architecture](/images/2-Proposal/IoT_Dashboard_Architecture.png)

*Figure 5-1. Current architecture with CloudFront/WAF/private S3, ALB/ASG FastAPI backends, RDS PostgreSQL Multi-AZ, YOLO UNO, and CloudWatch.*

The system operates through four main flows:

1. YOLO UNO reads sensors and submits telemetry directly through the ALB over HTTP.
2. FastAPI validates and stores telemetry in Amazon RDS for PostgreSQL.
3. The CloudFront-hosted dashboard uses `/api/*` through the ALB to retrieve data and create commands.
4. YOLO UNO polls for commands, controls the corresponding actuator, and sends an ACK; CloudWatch collects the related operational logs and metrics.

The AWS environment consists of **private Amazon S3, CloudFront, AWS WAF, ALB, target group, ASG, two EC2 instances with encrypted EBS, RDS PostgreSQL Multi-AZ, VPC/Security Groups, IAM, CloudWatch, and eight alarms**.

Begin with [Workshop Overview](5.1-Workshop-overview/).
