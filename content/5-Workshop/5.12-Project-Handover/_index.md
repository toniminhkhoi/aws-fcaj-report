---
title: "Project Handover"
date: "2026-07-28"
weight: 12
chapter: false
pre: " <b> 5.12. </b> "
---

## Overview and objectives

Transfer enough source, configuration, operational knowledge, and evidence for a new maintainer to start, validate, update, troubleshoot, and safely clean up the prototype.

## Repository structure

The application handover should contain:

```text
<application-repository>/
├── backend/              # FastAPI, Pydantic, SQLAlchemy, requirements
├── frontend/             # React, Vite, TypeScript, Tailwind CSS
├── hardware/             # PlatformIO firmware and YOLO UNO board definition
│   └── include/
│       └── secrets.example.h
└── README.md
```

The reviewed application source is maintained separately at `F:\aws-iot-dashboard`; this repository contains the Hugo report and Workshop. Record:

- source repository: `<SOURCE_REPOSITORY_URL>`;
- demo video: `<VIDEO_DEMO_URL>`;
- deployed application commit: `<COMMIT_SHA>`;
- AWS region/resource inventory location: `<HANDOVER_EVIDENCE_LOCATION>`.

## Start procedures

Backend on EC2 Linux Bash:

```bash
sudo systemctl start aws-iot-backend
sudo systemctl status aws-iot-backend --no-pager
curl -i http://127.0.0.1:8000/api/health
```

Frontend on Windows PowerShell:

```powershell
Set-Location .\aws-iot-dashboard\frontend
npm install
npm run dev
```

Hardware in a PlatformIO terminal:

```bash
pio run
pio run --target upload
pio device monitor --baud 115200
```

## Required local secrets

| Location | Values | Storage rule |
| :--- | :--- | :--- |
| Backend `.env` | `DATABASE_URL` and source-defined settings | EC2/local only; restricted; ignored |
| Firmware `secrets.h` | Wi-Fi, API URL, `room_01` | Local only; ignored |
| Frontend `.env.local` if used | API base URL | Local only; ignored |
| EC2 key | Private key | Approved local secret storage; never Git |

Handover the retrieval/rotation process, not plaintext credentials in the report.

## AWS and operational checklist

- [ ] Correct AWS account and region are known.
- [ ] VPC, public subnet, DB Subnet Group, route tables, and tags are recorded.
- [ ] EC2, EBS, key owner, IAM Role, and `iot-ec2-sg` are recorded.
- [ ] RDS identifier/endpoint, database `iot_dashboard`, and `iot-rds-sg` are recorded.
- [ ] RDS remains private and port 5432 is sourced from the EC2 SG.
- [ ] `aws-iot-backend` and CloudWatch Agent start at boot.
- [ ] Backend log group, metric dimensions, retention, and alarms are recorded.
- [ ] `room_01` firmware build, exact GPIO map, and safe power requirements are recorded.
- [ ] Latest T01-T15 results and open issues are linked.
- [ ] Cost owner and clean-up date are assigned.

## Update deployment procedure

In EC2 Linux Bash:

```bash
cd ~/aws-iot-dashboard
git status --short
git pull --ff-only
source backend/venv/bin/activate
pip install -r backend/requirements.txt
cd backend
python -m app.database.init_db
cd ..
sudo systemctl restart aws-iot-backend
sudo systemctl status aws-iot-backend --no-pager
curl -i http://127.0.0.1:8000/api/health
```

Review model/schema changes and release notes first. `app.database.init_db` uses SQLAlchemy `create_all`; it is not a migration engine, so destructive or incompatible schema changes require an explicit reviewed procedure. Record the previous/new commit and rollback procedure. Never discard local changes with `git reset --hard`.

## Database and CloudWatch checks

From EC2 Linux Bash:

```bash
psql "host=<RDS_ENDPOINT> port=5432 dbname=iot_dashboard user=<DB_USER> sslmode=require"
sudo systemctl status amazon-cloudwatch-agent --no-pager
sudo tail -n 100 /opt/aws/amazon-cloudwatch-agent/logs/amazon-cloudwatch-agent.log
```

In `psql`, run `\dt`, inspect `devices`, `telemetry_logs`, and `commands`, and use read-only validation queries. In CloudWatch, verify region, both backend log groups, recent timestamps, `IoTDashboard/EC2` guest metrics, native EC2 CPU, RDS CPU/connections, and the six documented alarm names/states.

## Known limitations

The documented prototype uses one room, direct HTTP on port 8000 for demo, a changeable EC2 public IP, periodic polling, and an uncalibrated analog light value. It has no implemented HTTPS, route authentication/API-key enforcement, HA, Multi-AZ proof, load balancer, rate limiting, or AI model. The frontend can simulate data and success after failures, stores mode locally, mislabels light as Lux, and hard-codes an EC2 target; the backend lacks command enum validation and strict ACK ownership checks. GPIO values, source paths/schema, and proposed alarm thresholds are recorded in sections 5.5, 5.6, and 5.9, but deployment evidence must still confirm the running environment.

## Team responsibilities

| Member | Responsibility | Contribution evidence |
| :--- | :--- | :--- |
| **Pham Le Minh Khoi** | AWS architecture, VPC, Security Groups, IAM Role, EC2, RDS, CloudWatch, DevOps, YOLO UNO hardware, sensors, actuators, telemetry, command polling, ACK | [5.3 architecture](../5.3-Architecture-and-Service-Design/), [5.4 AWS](../5.4-AWS-Infrastructure-Setup/), [5.6 hardware](../5.6-Hardware-Integration/), [5.9 monitoring](../5.9-CloudWatch-Monitoring/) |
| **Ngo Minh Thuan** | FastAPI backend, endpoints, Pydantic schemas, SQLAlchemy models, PostgreSQL integration, telemetry processing, command lifecycle, ACK processing | [5.3 API/data](../5.3-Architecture-and-Service-Design/), [5.5 backend/database](../5.5-Backend-and-Database/), [5.8 validation](../5.8-End-to-End-Testing/) |
| **Thuong Dinh Hung** | React + Vite frontend, dashboard UI, telemetry visualization, controls, overall integration, debugging, demo video recording/editing | [5.7 frontend](../5.7-Frontend-Integration/), [5.8 integration evidence](../5.8-End-to-End-Testing/), demo link recorded in repository structure |
| **Le Bao Khanh** | Documentation, proposal, blogs, weekly worklog, event reports, Workshop, bilingual review, navigation, screenshots, quality assurance | [5.1 rubric/output](../5.1-Workshop-overview/), [5.11 documentation/customization](../5.11-Results-Challenges-Future/), bilingual Workshop and Hugo QA |

The table preserves the agreed assignment and points reviewers to contribution evidence. It does not replace the separate [individual reflections in section 5.11](../5.11-Results-Challenges-Future/); each member must review and sign off both ownership and reflection before final submission.

## Final handover checklist

- [ ] Application source links and exact commit IDs open for the receiver.
- [ ] No credential appears in Git, screenshots, video, or this Workshop.
- [ ] Backend, frontend, and firmware start procedures were demonstrated.
- [ ] OpenAPI routes and database schemas were reviewed from source.
- [ ] Numeric GPIO map and power diagram were handed over.
- [ ] Test matrix contains actual evidence and status.
- [ ] Contribution evidence is attributable to the named member and the individual reflection was reviewed.
- [ ] CloudWatch configuration and alarm thresholds were confirmed.
- [ ] Open issues, limitations, owners, cost decision, and clean-up status were signed off.

<!-- TODO IMAGE: /images/5-Workshop/5.12-handover/repository-handover-checklist.png — Final redacted repository/resource/test handover checklist with commit IDs, owners, open issues, and team sign-off. -->

## Demo and Handover Resources

The final handover package includes:

- Source code repository
- End-to-end demo video
- Deployment and operation documentation
- Bilingual Workshop
- Architecture diagram
- AWS resource clean-up instructions

See [References]({{% relref "8-References/_index.md" %}}).

Return to the [Workshop landing page](../).
