---
title: "Project Handover"
date: "2026-07-28"
weight: 12
chapter: false
pre: " <b> 5.12. </b> "
---

## Overview and objectives

Transfer enough source, configuration, operational knowledge, and evidence for a new maintainer to start, validate, update, troubleshoot, and safely clean up the prototype.

## Handover Repository Structure

The main repository contains the backend and frontend source code, YOLO UNO firmware, architecture diagrams, and bilingual README documentation. The `main` branch is used as the final project handover version.

<p align="center">
  <img src="/images/5-Workshop/5.12-handover/repository-handover-checklist.png"
       alt="GitHub repository structure of the AWS IoT Monitoring and Control Dashboard project"
       width="100%" />
</p>

*Figure 22. Final project repository structure, including the backend, frontend, YOLO UNO firmware, architecture diagrams, and bilingual README files.*

The screenshot shows that the handover repository contains the source code for the main system components, bilingual README files, and architecture resources. Files containing private configuration values must be excluded through `.gitignore` and verified separately before handover.

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

## Final Handover Checklist

| Item | Status |
|---|---|
| FastAPI backend source code | Completed |
| React + Vite frontend source code | Completed |
| PlatformIO firmware for YOLO UNO | Completed |
| English and Vietnamese README files | Completed |
| Architecture diagram and hardware wiring documentation | Completed |
| End-to-end demo video | Completed |
| Bilingual Workshop documentation | Completed |
| Backend and hardware deployment instructions | Completed |
| Testing and CloudWatch instructions | Completed |
| AWS resource clean-up instructions | Completed |
| Secrets and private credentials excluded from Git | Verified |

### Secret Verification

Secrets and credentials are private values used to authenticate users, devices, applications, or cloud resources. These values must not be committed to the GitHub repository.

Files and values that must be excluded include:

- `backend/.env`
- `hardware/include/secrets.h`
- Wi-Fi SSIDs when they are not intended to be public
- Wi-Fi passwords
- Amazon RDS usernames and passwords
- `DATABASE_URL` values containing credentials
- AWS Access Key IDs
- AWS Secret Access Keys
- AWS session tokens
- SSH private keys such as `.pem` files
- API tokens
- GitHub personal access tokens
- Other private credentials

Template files may be committed:

- `.env.example`
- `secrets.example.h`

However, templates must contain placeholders instead of real credentials.

Examples:

```env
DATABASE_URL=postgresql://USERNAME:PASSWORD@RDS_ENDPOINT:5432/DATABASE_NAME
```

```cpp
#define WIFI_SSID "YOUR_WIFI_SSID"
#define WIFI_PASSWORD "YOUR_WIFI_PASSWORD"
#define API_BASE_URL "http://YOUR_EC2_ADDRESS"
```

> Removing a secret from the latest version does not necessarily remove it from the Git history. If a secret was previously committed, revoke or rotate the credential and clean the repository history when required.

### Pre-Handover Verification

Run the following commands from the repository root.

Check the current working tree:

```bash
git status
```

List all tracked files:

```bash
git ls-files
```

Search the commit history for sensitive files:

```bash
git log --all --oneline -- .env secrets.h "*.pem"
```

Search tracked files for common credential variable names:

```bash
git grep -n -I -E "AWS_ACCESS_KEY_ID|AWS_SECRET_ACCESS_KEY|AWS_SESSION_TOKEN|DATABASE_URL|WIFI_PASSWORD|PRIVATE_KEY|API_TOKEN"
```

Check whether sensitive files are currently tracked:

```bash
git ls-files | grep -E '(^|/)\.env$|secrets\.h$|\.pem$'
```

Notes:

- Do not publish command output containing real credentials.
- `git status` only checks the current working tree.
- `git log --all` helps review the complete commit history.
- If an AWS key, password, or token was committed, revoke or rotate it.
- Deleting a file without rotating the exposed credential is insufficient.

## Handover Resources

The final project handover package includes:

- GitHub source code repository.
- End-to-end demo video.
- Backend deployment documentation.
- YOLO UNO firmware instructions.
- Bilingual Workshop documentation.
- AWS architecture diagram.
- Testing and CloudWatch monitoring instructions.
- AWS resource clean-up instructions.

The source code, demo video, and related documents are collected in the [References]({{% relref "8-References/_index.md" %}}) section.

Return to the [Workshop landing page](../).
