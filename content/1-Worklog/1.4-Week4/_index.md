---
title: "Week 4 - Backend and Database Foundation"
date: "2026-06-22"
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

> **Period:** 22–28 June 2026
> **Role:** Managed the EC2 runtime and service operations and supported the backend member with PostgreSQL connectivity.

## Objectives

- Prepare the Python environment and deploy the FastAPI structure on EC2.
- Connect the backend to the `iot_dashboard` database.
- Operate Uvicorn reliably through `systemd`.

## Work completed

| Period | Activity | Recorded outcome |
| :--- | :--- | :--- |
| 22 June | Installed Git, Python, the PostgreSQL client, and required utilities on Amazon Linux | Prepared EC2 to run and diagnose the backend |
| 23 June | Cloned the application repository, created `venv`, and installed `requirements.txt` | Confirmed the Uvicorn entry point as `main:app` |
| 24 June | Created a local `.env` with a sanitized `DATABASE_URL` | Connected to RDS without committing credentials |
| 25–26 June | Ran `app.database.init_db` and inspected PostgreSQL | Created `devices`, `telemetry_logs`, and `commands` through SQLAlchemy `create_all` |
| 27–28 June | Created `aws-iot-backend.service`, configured log files, and tested `/api/health` | The service reached `active (running)` and the health check returned HTTP 200 |

## Weekly outcomes

- Operated FastAPI on EC2 under `systemd`.
- Connected the backend to the private RDS database.
- Confirmed that the project did not use Alembic migrations and avoided documenting Alembic as implemented.

## Challenges and lessons learned

Linux user names, paths, environment files, and the Uvicorn module must match between the shell and service unit. `journalctl`, application logs, and the health endpoint provide complementary diagnostic evidence.

## Workshop references

- [5.5 Backend Deployment and Database Integration](../../5-workshop/5.5-backend-and-database/)
- [5.12 Project Handover](../../5-workshop/5.12-project-handover/)
