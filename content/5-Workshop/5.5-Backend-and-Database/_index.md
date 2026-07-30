---
title: "Backend Deployment and Database Integration"
date: "2026-07-28"
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

## Overview and objectives

Deploy the FastAPI backend in a Python virtual environment on Amazon EC2, manage it with `aws-iot-backend.service`, and connect it to the `iot_dashboard` database on Amazon RDS for PostgreSQL. The runbook uses `ec2-user`, `/home/ec2-user/aws-iot-dashboard/backend`, virtual environment `venv`, and Uvicorn entry point `main:app`.

## Step 1 - Deploy the FastAPI Backend

Connect from Windows PowerShell, install the required packages, clone the repository, and create the virtual environment:

```powershell
ssh -i "$env:USERPROFILE\.ssh\<KEY_FILE>.pem" ec2-user@<EC2_PUBLIC_IP>
```

```bash
sudo dnf update -y
sudo dnf install -y git python3 python3-pip postgresql15 curl
git clone <REPOSITORY_URL> ~/aws-iot-dashboard
cd ~/aws-iot-dashboard/backend
python3 -m venv venv
source venv/bin/activate
python -m pip install --upgrade pip
pip install -r requirements.txt
```

Create an ignored `.env` file. Do not commit credentials:

```dotenv
DATABASE_URL=postgresql://<DB_USER>:<DB_PASSWORD>@<RDS_ENDPOINT>:5432/iot_dashboard?sslmode=require
```

A manual start can be used for the first deployment check:

```bash
source venv/bin/activate
uvicorn main:app --host 0.0.0.0 --port 8000
```

## Step 2 - Configure the systemd Service

Create `/etc/systemd/system/aws-iot-backend.service`:

```ini
[Unit]
Description=AWS IoT FastAPI backend
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=ec2-user
WorkingDirectory=/home/ec2-user/aws-iot-dashboard/backend
EnvironmentFile=/home/ec2-user/aws-iot-dashboard/backend/.env
ExecStart=/home/ec2-user/aws-iot-dashboard/backend/venv/bin/uvicorn main:app --host 0.0.0.0 --port 8000
Restart=on-failure
RestartSec=5
StandardOutput=append:/var/log/aws-iot-backend/backend.log
StandardError=append:/var/log/aws-iot-backend/backend-error.log

[Install]
WantedBy=multi-user.target
```

Prepare the log directory, enable the service, and start it:

```bash
sudo install -d -o ec2-user -g ec2-user /var/log/aws-iot-backend
sudo systemctl daemon-reload
sudo systemctl enable aws-iot-backend
sudo systemctl restart aws-iot-backend
```

Once enabled, systemd starts the backend when EC2 boots; Uvicorn does not need to be started manually after every restart.

## Step 3 - Verify the Backend Service

Run:

```bash
sudo systemctl status aws-iot-backend --no-pager -l
curl http://127.0.0.1:8000/api/health
```

<p align="center">
  <img src="/images/5-Workshop/5.5-backend-database/backend-systemd-health-check.png"
       alt="FastAPI backend systemd service and health check"
       width="100%" />
</p>

*Figure 8. The `aws-iot-backend.service` is `active (running)`, and the `/api/health` endpoint returns an `ok` status.*

Figure 8 shows that the unit was loaded by systemd, the service is `active (running)`, and Uvicorn is the main process. The health endpoint also returns valid JSON with `"status":"ok"`. Together, these observations provide evidence that the backend is deployed and can accept a local HTTP request. They do not establish High Availability or guarantee failure-free operation.

## Step 4 - Connect EC2 to Amazon RDS

From EC2, connect with SSL/TLS required:

```bash
psql "host=<RDS_ENDPOINT> port=5432 dbname=iot_dashboard user=<DB_USER> sslmode=require"
```

In `psql`, confirm the active database and connection information without exposing the password:

```sql
SELECT current_database(), current_user;
\conninfo
\dt
```

If the schema has not been initialized, run the source-defined command from the backend virtual environment, then verify it again:

```bash
cd ~/aws-iot-dashboard/backend
source venv/bin/activate
python -m app.database.init_db
```

## Step 5 - Verify PostgreSQL Tables and Commands

The deployed `iot_dashboard` database shown in the evidence contains `commands`, `devices`, `sensor_readings`, and `telemetry_logs`. Query the most recent command records:

```sql
SELECT id, device_id, command, state, timestamp
FROM commands
ORDER BY id DESC
LIMIT 6;
```

<p align="center">
  <img src="/images/5-Workshop/5.5-backend-database/postgresql-tables-and-commands.png"
       alt="PostgreSQL tables and executed IoT commands"
       width="100%" />
</p>

*Figure 9. The EC2-to-Amazon RDS PostgreSQL connection, database tables, and recent commands in the `Executed` state.*

The screenshot confirms an SSL/TLS PostgreSQL session from EC2 to the `iot_dashboard` database. It lists the four application tables and recent command rows whose `device_id` is `room_01`. Examples include `CURTAIN_CLOSE`, `CURTAIN_OPEN`, `MODE_AUTO`, and `LIGHT_OFF`, all displayed in the `Executed` state. Database credentials are not shown.

## Step 6 - Expected Results

- `aws-iot-backend.service` is enabled and `active (running)`.
- Uvicorn is the main backend process.
- `GET /api/health` returns JSON with status `ok`.
- EC2 connects to Amazon RDS for PostgreSQL using SSL/TLS.
- `commands`, `devices`, `sensor_readings`, and `telemetry_logs` appear in `psql`.
- The query returns command records whose `device_id` is `room_01`.
- No password, access key, or other credential appears in commands or screenshots.

## Troubleshooting

| Symptom | Diagnosis and correction |
| :--- | :--- |
| Backend service fails | Inspect `systemctl status` and `journalctl -u aws-iot-backend`; verify the user, paths, `.env`, and Uvicorn module |
| Port 8000 is already in use | Run `sudo ss -ltnp \| grep :8000` and stop the unintended process |
| Health check is refused | Confirm the service is running and Uvicorn is listening on port 8000 |
| RDS connection times out | Check the RDS endpoint, Region, subnet path, and Security Group source on port 5432 |
| PostgreSQL authentication fails | Verify the database name, user, password encoding, and the `.env` loaded by systemd |
| SSL connection fails | Confirm the endpoint hostname and selected `sslmode`; use the required RDS CA bundle when certificate verification is enabled |
| Tables are missing | Run the source-defined initialization process and inspect the correct `iot_dashboard` database |

Next: [integrate YOLO UNO hardware](../5.6-Hardware-Integration/).
