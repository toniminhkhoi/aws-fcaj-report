---
title: "Proposal"
date: "2026-06-15"
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# AWS IoT Monitoring and Control Dashboard

### Downloadable version: <a href="/files/2-Proposal/IoT_Dashboard_Proposal.pdf" download>IoT Dashboard Proposal (PDF)</a>

> **PDF update required:** the downloadable PDF is retained for reference and must be re-exported to match this revised web proposal before final submission.

---

## 1. Executive Summary

The **AWS IoT Monitoring and Control Dashboard** addresses the need to monitor room conditions and control physical devices from a centralized interface. It connects YOLO UNO hardware to a backend deployed on AWS so that users can view temperature, humidity, and light readings, issue remote commands, and verify physical execution through command acknowledgement.

In the current Workshop deployment:

- YOLO UNO collects environmental telemetry and controls the fan, light, and curtain servo;
- FastAPI runs on Amazon EC2 under `aws-iot-backend.service`;
- Amazon RDS for PostgreSQL stores telemetry and command states;
- a local React + Vite frontend presents near-real-time data, history, controls, and rule-based recommendations;
- the device polls for commands and sends an ACK after execution; and
- Amazon CloudWatch collects backend logs, operational metrics, and alarm states.

The result is a reproducible Smart Room prototype that demonstrates an end-to-end flow from physical sensing to cloud storage, dashboard visualization, remote control, and physical-device acknowledgement.

---

## 2. Problem Statement and Target Users

### 2.1 Problem Statement

Small classrooms, laboratories, and equipment rooms often rely on sensors and actuators that operate independently. Values may only be visible at the device, historical records are not stored centrally, and operators cannot easily inspect room conditions or control equipment remotely. A dashboard action alone also does not prove that a fan, light, or curtain physically completed the requested operation.

Troubleshooting is fragmented when application logs, infrastructure metrics, database records, and device states are not connected. This project creates one traceable workflow for telemetry, commands, acknowledgement, and monitoring.

### 2.2 Target Users

| Target user | Main need |
|---|---|
| Room or laboratory manager | Monitor environmental conditions and actuator status from one interface |
| Lecturers and students | Observe telemetry, historical data, and an end-to-end IoT experiment |
| Operators | Remotely control the fan, light, and curtain |
| Development team | Debug through logs, metrics, APIs, database records, and command states |

### 2.3 Delivered Value

- A centralized dashboard for current values, historical telemetry, and controls.
- Two-way communication between the cloud backend and physical hardware.
- Persistent telemetry and command history in PostgreSQL.
- Physical-action verification through `Pending` and `Executed` states.
- Centralized logs and metrics for troubleshooting.
- A documented foundation that can later support additional rooms or devices.

---

## 3. Objectives and Scope

### 3.1 Hardware Objectives

- Read temperature and humidity from the DHT20 and read the analog light sensor.
- Control the fan, light/relay, and curtain servo; display status on LCD1602 I2C.
- Connect through Wi-Fi and HTTP REST.
- Support eight commands: `MODE_AUTO`, `MODE_MANUAL`, `FAN_ON`, `FAN_OFF`, `LIGHT_ON`, `LIGHT_OFF`, `CURTAIN_OPEN`, and `CURTAIN_CLOSE`.

### 3.2 Backend and Cloud Objectives

- Deploy FastAPI, Uvicorn, SQLAlchemy, and Pydantic on Amazon EC2.
- Manage the backend through `aws-iot-backend.service`.
- Store telemetry and commands in the `iot_dashboard` database on RDS PostgreSQL.
- Configure VPC networking, routing, and Security Groups for the deployment.
- Keep RDS non-public and allow PostgreSQL only from the EC2 Security Group.
- Use an EC2 IAM Role/Instance Profile for CloudWatch Agent permissions.

### 3.3 Frontend Objectives

- Display temperature, humidity, light, and historical charts through REST polling.
- Create fan, light, curtain, Manual, and Auto commands.
- Present threshold-based, rule-based analysis without describing it as machine learning.

### 3.4 Monitoring and Documentation Objectives

- Send backend logs to CloudWatch Logs.
- Monitor EC2 CPU, memory, disk, RDS CPU, and database connections.
- Configure alarms for important operational thresholds.
- Deliver bilingual Workshop instructions, source code, architecture, demo, and clean-up guidance.

### 3.5 Current Scope

The prototype uses `room_01` as the logical identifier of the model room. It covers three environmental measurements, three physical actuators, HTTP REST, command polling, ACK, one EC2 instance, one Single-AZ RDS PostgreSQL instance, a local frontend, and CloudWatch monitoring.

### 3.6 Out of Scope

The current delivery does not include a production multi-room rollout, user authentication, HTTPS with a custom domain, automatic horizontal scaling, cross-AZ disaster recovery, a mobile application, an automated delivery pipeline, or a trained machine-learning model. These capabilities require a separate review of requirements, cost, security, and operating complexity.

---

## 4. Solution Architecture

![AWS IoT Monitoring and Control Dashboard architecture](/images/2-Proposal/IoT_Dashboard_Architecture.png)

*Current deployment architecture of the AWS IoT Monitoring and Control Dashboard, including the local frontend, YOLO UNO, FastAPI on EC2, RDS PostgreSQL, and Amazon CloudWatch.*

### 4.1 Resource Placement

| Location | Components | Responsibility |
|---|---|---|
| Outside AWS | Dashboard user, local React + Vite frontend, YOLO UNO ESP32-S3 | User interaction, telemetry, command execution, and ACK |
| AWS Cloud, outside VPC | EC2 IAM Role/Instance Profile, CloudWatch | Permissions, logs, metrics, dashboard, and alarms |
| Amazon VPC | Internet Gateway, public route table, public application subnet, private DB subnet | Network boundary and routing |
| Public application subnet | Amazon EC2 | FastAPI backend and CloudWatch Agent |
| Private DB subnet through DB Subnet Group | Amazon RDS for PostgreSQL | Private telemetry and command persistence |
| Attached to EC2 in the same AZ | 10 GiB gp3 EBS root volume | OS, application, and runtime files |

CloudWatch Agent runs inside EC2 and sends backend logs plus memory and disk metrics to CloudWatch. RDS publishes managed metrics to CloudWatch. EBS is the EC2 root volume, not an independent service inside a subnet.

### 4.2 Telemetry Flow

1. YOLO UNO reads the DHT20 and analog light sensor.
2. Firmware sends `POST /api/telemetry` to FastAPI on EC2.
3. FastAPI validates and stores the record in RDS PostgreSQL.
4. The frontend polls latest and history endpoints.
5. The dashboard displays current readings and ordered history.

### 4.3 Command and Acknowledgement Flow

1. The operator creates a command from the frontend.
2. FastAPI stores it in the `Pending` state.
3. YOLO UNO polls the latest command for `room_01`.
4. Firmware executes the actuator or mode command.
5. YOLO UNO sends an ACK with the numeric command ID.
6. FastAPI updates the matching record to `Executed`.

### 4.4 Main REST Endpoints

| Method | Endpoint | Purpose |
|---|---|---|
| `GET` | `/api/health` | Verify backend health |
| `POST` | `/api/telemetry` | Store telemetry |
| `GET` | `/api/devices/{device_id}/latest` | Retrieve latest telemetry |
| `GET` | `/api/devices/{device_id}/history` | Retrieve telemetry history |
| `POST` | `/api/devices/{device_id}/commands` | Create a command |
| `GET` | `/api/devices/{device_id}/commands/latest` | Poll the latest command |
| `POST` | `/api/devices/{device_id}/commands/{command_id}/ack` | Acknowledge execution |

### 4.5 Security Boundary

- RDS has no public Internet access.
- RDS TCP `5432` accepts traffic from the EC2 Security Group, not a public CIDR.
- EC2 uses an IAM Role/Instance Profile for CloudWatch instead of static AWS credentials.
- Runtime values use environment or local secret files.
- `backend/.env`, `hardware/include/secrets.h`, `*.pem`, passwords, and credentials must not be committed.

---

## 5. Technical Implementation Plan

### Phase 1 — Requirements and AWS Foundation

Confirm the Smart Room functional requirements and acceptance criteria, assign team responsibilities, and define `room_01` as the logical identifier of the model room. Then design the AWS infrastructure, deploy EC2 and RDS, and verify private database connectivity.

### Phase 2 — Backend and Database

Define the PostgreSQL schema; implement health, telemetry, latest, history, command, polling, and ACK endpoints; configure environment values and systemd.

### Phase 3 — Hardware and Firmware

Connect DHT20, the light sensor, fan, light/relay, servo, and LCD; develop PlatformIO firmware for sensing, Wi-Fi, telemetry, polling, actuator control, modes, and ACK; validate all eight commands.

### Phase 4 — Frontend and Integration

Build the React + Vite + TypeScript dashboard, history charts, controls, and rule-based recommendations; integrate all components and resolve API, CORS, command-state, and hardware issues.

### Phase 5 — Monitoring, Validation, and Handover

Configure CloudWatch Agent, logs, metrics, and five alarms; run end-to-end tests; review security; complete the bilingual Workshop, evidence, demo, clean-up guide, and handover.

---

## 6. Timeline and Milestones

The Proposal follows the eight-week Worklog from **01/06/2026 to 31/07/2026**.

| Week | Period | Main activities | Milestone |
|---|---|---|---|
| **Week 1** | 01/06–07/06 | Requirements, scope, initial architecture, roles, and plan | Agreed functions, acceptance scope, architecture, and assignments |
| **Week 2** | 08/06–14/06 | AWS architecture, VPC, IAM, and Security Groups | Reviewed cloud and security design |
| **Week 3** | 15/06–21/06 | EC2, EBS, RDS, network, and database connectivity | Running EC2 and available private PostgreSQL |
| **Week 4** | 22/06–28/06 | FastAPI, schema, environment, and systemd | Backend running and connected to RDS |
| **Week 5** | 29/06–05/07 | Telemetry, latest, history, command polling, and ACK APIs | Complete REST flow and persisted command states |
| **Week 6** | 06/07–12/07 | YOLO UNO firmware, sensors, actuators, Wi-Fi, and REST | Hardware sends telemetry and processes eight commands |
| **Week 7** | 13/07–19/07 | React dashboard, charts, controls, integration, and debugging | Frontend reads AWS data and creates commands |
| **Week 8** | 20/07–31/07 | End-to-end tests, CloudWatch, security, documentation, demo, and handover | Verified evidence and submission-ready bilingual documentation |

---

## 7. Resource Configuration, Cost and Optimization

### 7.1 Current Resource Configuration

| Resource | Current configuration | Main cost factor | Optimization |
|---|---|---|---|
| EC2 | `t3.micro`, Linux, Public IPv4, Single-AZ | Running hours and data transfer | Stop when unused; inspect CPU/memory before resizing |
| EBS | 10 GiB `gp3` root volume | Provisioned storage and snapshots | Keep required storage and remove unused snapshots |
| RDS PostgreSQL | `db.t4g.micro`, Single-AZ, non-public | Instance hours, storage, backups, transfer | Stop during permitted idle periods; review backups and connections |
| RDS storage | 20 GiB General Purpose SSD | Provisioned storage | Monitor growth and avoid unnecessary retention |
| CloudWatch | Logs, EC2/RDS metrics, dashboard, five alarms | Log ingestion/retention, custom metrics, dashboard, alarms | Set suitable retention and remove unused artifacts |
| Data transfer | HTTP telemetry and polling | Internet and data volume | Use reasonable intervals and compact payloads |
| IAM/VPC foundation | Role, VPC, subnets, route, IGW, Security Groups | Basic configuration usually has no direct hourly cost; attached resources and processing may cost | Remove unused dependencies carefully |

No fixed monthly total is claimed because charges depend on runtime, Region, traffic, retention, backups, and account pricing. Actual charges must be checked in AWS Billing and Cost Management.

### 7.2 Optimization Actions

- Stop EC2/RDS when the demo environment is not required and policy permits it.
- Use CloudWatch evidence to assess resource sizing.
- Retain logs only as long as needed for evaluation and troubleshooting.
- Balance polling responsiveness against request volume.
- Follow the dependency-aware clean-up order in Workshop section 5.10.

---

## 8. Risk Assessment

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| EC2 Public IPv4 changes after stop/start | Medium | High | Update frontend/firmware and document the active endpoint |
| Workshop uses HTTP | High | High | Avoid sensitive payloads, limit exposure, and plan HTTPS before broader use |
| Security Group rules are too broad | Medium | High | Restrict SSH; keep RDS `5432` limited to the EC2 Security Group |
| Credentials are committed | Low | High | Use local secret files, placeholder examples, and review Git status |
| EC2 cannot reach RDS | Medium | High | Verify subnet, DB Subnet Group, DNS, SG rules, and TLS with `psql` |
| Device loses Wi-Fi/backend connectivity | Medium | Medium | Add retry/reconnect logic and show connection state |
| Polling creates excess requests | Medium | Medium | Use documented intervals and review browser/backend logs |
| Command remains `Pending` | Medium | High | Check polling, command name/ID, actuator result, ACK, and DB state |
| GPIO/power error causes instability | Medium | High | Use verified pin map, shared ground, safe power, and incremental tests |
| Monitoring evidence is incomplete | Medium | Medium | Verify Agent, log group, metric dimensions, retention, and five alarms |
| Resources continue generating cost | Medium | High | Assign clean-up responsibility and follow dependency order |

---

## 9. Expected Results and Success Criteria

### 9.1 Expected Results

The prototype should provide a traceable path from physical readings to AWS storage and dashboard visualization, plus a return path from dashboard commands to actuator execution and ACK. Evidence must allow another reviewer to inspect the application, database, hardware, and monitoring layers.

### 9.2 Measurable Success Criteria

| Success criterion | Acceptance evidence |
|---|---|
| Backend runs | `systemctl status aws-iot-backend` shows `active (running)` |
| Health works | `/api/health` returns `status: ok` |
| EC2 connects to RDS | `psql` uses SSL/TLS and `\dt` shows application tables |
| Telemetry persists | PostgreSQL contains a `room_01` telemetry record |
| Dashboard displays data | Temperature, humidity, and light show Live AWS |
| History works | Charts use `/api/devices/room_01/history` |
| Frontend calls succeed | Latest, history, and command requests return HTTP `200` |
| Commands persist | `commands` contains the created command and state |
| Actuators respond | Fan, light, and curtain servo react in demo evidence |
| ACK lifecycle works | The same command changes `Pending` → `Executed` |
| CloudWatch Logs works | Backend log stream contains FastAPI request logs |
| Metrics are visible | EC2 CPU/disk, RDS CPU/connections, and Agent-collected EC2 memory are shown |
| Alarms exist | CloudWatch lists five project alarms |
| RDS is private | Internet access is disabled or `Publicly accessible: No` |
| PostgreSQL is restricted | RDS accepts TCP `5432` from the EC2 Security Group |
| Secrets stay out of Git | `.env`, `secrets.h`, `*.pem`, and real credentials are untracked |

---

## 10. Current Limitations and Future Improvements

### 10.1 Current Limitations

- One EC2 with Public IPv4 and one Single-AZ RDS instance.
- Local frontend outside AWS; HTTP and no user authentication in the Workshop.
- Polling-based command delivery and dashboard refresh.
- One model room identified by `room_01`.
- Rule-based recommendations, not trained machine learning.
- Manual deployment and testing.

### 10.2 Future Improvements

After reviewing requirements, security, cost, and complexity, future work may add HTTPS and authentication, tighter network boundaries, multiple rooms and identities, availability/recovery procedures, event-driven communication at greater scale, automated testing/deployment, stronger secret and backup handling, operational notifications, and a mobile-friendly experience.

These are future options and are not part of the current deployment.

---

## 11. Team Responsibilities

| Member | Role and responsibilities |
|---|---|
| **Phạm Lê Minh Khôi** | AWS infrastructure, EC2, EBS, RDS, VPC, Security Groups, IAM, CloudWatch, DevOps, PlatformIO firmware development, and YOLO UNO hardware integration |
| **Thượng Đình Hưng** | React + Vite frontend, dashboard, overall integration, debugging, and demo recording support |
| **Ngô Minh Thuận** | FastAPI backend, REST endpoints, PostgreSQL integration, telemetry, command processing, and ACK handling |
| **Lê Bảo Khánh** | Documentation and QA, Proposal, Blogs, Worklog, Event Reports, and bilingual Workshop content |

---

## 12. Deliverables and Evidence

| Deliverable | Content | Evidence |
|---|---|---|
| GitHub repository | Backend, frontend, hardware, diagrams, bilingual README | [Source Code]({{% relref "8-References/8.1-source-code/_index.md" %}}), public `main` |
| FastAPI backend | Health, telemetry, history, command, polling, ACK | [Backend Workshop]({{% relref "5-Workshop/5.5-Backend-and-Database/_index.md" %}}), systemd and health evidence |
| PostgreSQL database | Tables, telemetry, command states | `psql`, `\dt`, telemetry and command queries |
| React + Vite frontend | Telemetry, controls, rules, history charts | [Frontend Workshop]({{% relref "5-Workshop/5.7-Frontend-Integration/_index.md" %}}), screenshots |
| YOLO UNO firmware | Sensors, actuators, modes, polling, ACK | [Hardware Workshop]({{% relref "5-Workshop/5.6-Hardware-Integration/_index.md" %}}), source, build, demo |
| End-to-end validation | Telemetry persistence, lifecycle, physical execution | [Testing Workshop]({{% relref "5-Workshop/5.8-End-to-End-Testing/_index.md" %}}), [Demo Video]({{% relref "8-References/8.2-demo-video/_index.md" %}}) |
| CloudWatch monitoring | Logs, metrics, dashboard, five alarms | [CloudWatch Workshop]({{% relref "5-Workshop/5.9-CloudWatch-Monitoring/_index.md" %}}), screenshots |
| Bilingual Workshop | Deployment, integration, testing, monitoring, handover | [Workshop]({{% relref "5-Workshop/_index.md" %}}) |
| Clean-up guide | Inventory, cost, security, dependency-aware removal | [Cost, Security, Clean-up]({{% relref "5-Workshop/5.10-Cost-Security-Cleanup/_index.md" %}}) |
| Reference set | Source, demo, documents, technical links | [References]({{% relref "8-References/_index.md" %}}) |

Public project links:

- [Project source code](https://github.com/toniminhkhoi/aws-iot-dashboard/tree/main)
- [End-to-end demo video](https://drive.google.com/file/d/1T97dUY58hbT2ppxvg7ESR12Jg9BA828W/view?usp=sharing)