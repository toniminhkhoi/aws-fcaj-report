---
title: "Workshop Overview"
date: "2026-07-28"
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

## Context and Problem

A Smart Room needs to retain environmental data for historical review, support remote device control, and confirm that each command was actually executed. If the sensors, dashboard, and actuators operate independently, operators cannot trace the complete data flow or distinguish between “the API accepted the command” and “the physical device completed the command.”

This Workshop builds one traceable workflow for a Smart Room identified by `device_id=room_01`. CloudFront and WAF deliver the private-S3 React frontend and forward browser `/api/*` traffic to an ALB. The ALB routes to two ASG-managed FastAPI instances; RDS PostgreSQL Multi-AZ stores telemetry and command states. YOLO UNO sends telemetry and polls commands directly through the ALB, executes them, and sends an ACK.

## Target Users and Proposed Solution

| Target user | Need | Workshop value |
| :--- | :--- | :--- |
| FCAJ participant / AWS learner | Deploy and validate an end-to-end workload on AWS | A reproducible path from infrastructure setup to application, hardware, monitoring, and evidence |
| Smart Room operator | Review current and historical readings and control room devices | One dashboard for telemetry, operating mode, fan, light, and curtain control |
| Maintainer / developer | Trace faults across the application, database, network, and hardware | Correlated evidence from browser requests, FastAPI logs, SQL queries, `systemd`, Serial Monitor, and CloudWatch |
| Project reviewer / FCAJ mentor | Assess AWS relevance, implementation depth, and individual contributions | Explicit architecture decisions, measurable tests, evidence links, and handover materials |

## Relevance to FCAJ and AWS

The Workshop reflects FCAJ learning outcomes by combining cloud architecture, Linux operations, networking, security, database integration, full-stack development, physical IoT, testing, monitoring, documentation, and project handover in one traceable implementation.

Each AWS service has a clear operational role: private S3, CloudFront, and WAF provide the web edge; ALB and Auto Scaling distribute backend traffic across two EC2 instances with encrypted EBS; RDS PostgreSQL Multi-AZ provides managed persistence; VPC/Security Groups enforce the ALB → backend → database path; an IAM Role authorizes monitoring; and CloudWatch collects logs, metrics, and alarm states.

## Technical Objectives

1. Receive telemetry from YOLO UNO for the Smart Room identified by `device_id=room_01`.
2. Retrieve the latest record and time-ordered telemetry history.
3. Support all `8` firmware commands for operating mode, fan, light, and curtain control.
4. Make command completion observable through the `Pending` → `Executed` transition and ACK.
5. Run FastAPI as a `systemd` service on ASG instances and monitor ALB, ASG, EC2, and RDS with CloudWatch.
6. Provide a reproducible bilingual Workshop, test evidence, and project handover package.

## Current Scope

| Component | Implemented scope |
| :--- | :--- |
| Smart Room identity | One room identified by `device_id=room_01` |
| Sensors | DHT20 temperature/humidity readings and raw analog light readings |
| Actuators and display | Fan, light/relay, curtain servo, and LCD1602 status display |
| Device software | PlatformIO firmware for YOLO UNO with Auto/Manual modes and `8` supported commands |
| Backend and data | ALB → ASG with two FastAPI instances; telemetry and commands in RDS PostgreSQL Multi-AZ |
| Frontend | React + Vite build in private S3, delivered by CloudFront OAC with WAF monitoring |
| Communication | Browser HTTPS to CloudFront; `/api/*` to ALB; device HTTP directly to ALB; polling, execution, and ACK |
| Monitoring | Backend logs, ALB/ASG/EC2/RDS dashboard widgets, and eight CloudWatch alarms |

## Functional Contract

| Capability | Observable result |
| :--- | :--- |
| Telemetry ingestion | A valid request creates one identifiable telemetry record in PostgreSQL |
| Latest telemetry | The latest record for `device_id=room_01` is returned |
| Telemetry history | Time-ordered records for `device_id=room_01` are returned |
| Fan control | `FAN_ON` and `FAN_OFF` are accepted and executed |
| Light control | `LIGHT_ON` and `LIGHT_OFF` are accepted and executed |
| Curtain control | `CURTAIN_OPEN` and `CURTAIN_CLOSE` are accepted and executed |
| Operating mode | `MODE_AUTO` enables firmware threshold control; `MODE_MANUAL` enables direct control |
| Command lifecycle | A new command is `Pending`; a successful ACK changes it to `Executed` |
| CloudWatch monitoring | Configured logs and metrics are visible, and alarms evaluate their thresholds |

The project uses rule-based logic rather than an AI model. In Auto mode, the firmware controls the fan when `temperature >= 30 °C`, the light when the raw analog value is `< 350`, and the curtain around the `< 700` threshold. A direct actuator command switches the firmware to Manual mode.

## Concrete Outputs

| Output | Artifact or evidence |
| :--- | :--- |
| AWS infrastructure | CloudFront/WAF/private S3, ALB/target group/ASG/EC2/EBS, RDS Multi-AZ, VPC/Security Groups, IAM, and CloudWatch evidence |
| Running backend | Active `aws-iot-backend` service and an HTTP 200 response from `GET /api/health` |
| PostgreSQL persistence | Table and query evidence for `devices`, `telemetry_logs`, and `commands` |
| YOLO UNO integration | GPIO/wiring documentation, successful PlatformIO build, telemetry flow, command execution, and ACK evidence |
| Dashboard | Latest/history data, control actions, command state, analytical recommendations, and physical-control demonstration |
| Monitoring | Backend logs, ALB/ASG/EC2/RDS metric widgets, and eight CloudWatch alarm configurations |
| Validation | T01–T14 test matrix with all results recorded as Pass and linked evidence |
| Handover | Source repository, bilingual README and Workshop, reference links, and final handover checklist |

## Measurable Success Criteria

| ID | Criterion | Measurement | Status |
| :--- | :--- | :--- | :--- |
| S01 | Backend availability | `GET /api/health` returns HTTP 200 from the deployed service | Pass |
| S02 | Telemetry persistence | One valid POST for `device_id=room_01` creates one identifiable row in `telemetry_logs` | Pass |
| S03 | Data retrieval | Latest and history requests return the stored `room_01` data in the expected order | Pass |
| S04 | Physical control | All six direct actuator commands are validated with command and physical evidence | Pass |
| S05 | Mode control | `MODE_AUTO` and `MODE_MANUAL` are verified through firmware behavior | Pass |
| S06 | Command completion | The same command ID is captured as `Pending` and later as `Executed` after ACK | Pass |
| S07 | Monitoring | Backend logs, ALB/ASG/EC2/RDS widgets, and eight alarm configurations are visible | Pass |
| S08 | Reproducibility and safety | T01–T14 are recorded as Pass, handover materials are available, and credentials are excluded from the repository | Pass |

The evidence supporting these criteria is provided in Sections 5.5–5.9 and 5.12.

## Troubleshooting Checkpoint

If the failing component is unclear, trace one request through the browser Network panel, FastAPI logs, PostgreSQL records, the device Serial Monitor, and the ACK state. Record the evidence before marking a test as passed.

Next: [prepare the required account, tools, and hardware](../5.2-Prerequisites/).
