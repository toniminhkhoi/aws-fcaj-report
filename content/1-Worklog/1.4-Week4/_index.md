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

| Workstream | Work completed | Result/Evidence |
| :--- | :--- | :--- |
| EC2 preparation | Installed Git, Python, the PostgreSQL client, and required utilities on Amazon Linux | EC2 had the environment needed to run and diagnose the backend |
| Application deployment | Cloned the repository, created `venv`, installed `requirements.txt`, and confirmed `main:app` | FastAPI ran through Uvicorn on EC2 |
| Configuration management | Created a local `.env` with `DATABASE_URL` and excluded secrets from Git | Backend connected to RDS without hard-coded credentials |
| Database initialization | Ran `app.database.init_db` and inspected the schema with the PostgreSQL client | Created `devices`, `telemetry_logs`, and `commands` through SQLAlchemy `create_all` |
| Backend operation | Created `aws-iot-backend`, configured logs, restarted the service, and tested `/api/health` | Service reached `active (running)` and health returned HTTP 200 |
## Weekly outcomes

- Operated FastAPI on EC2 under `systemd`.
- Connected the backend to the private RDS database.
- Confirmed that the project did not use Alembic migrations and avoided documenting Alembic as implemented.

## Challenges and lessons learned

Linux user names, paths, environment files, and the Uvicorn module must match between the shell and service unit. `journalctl`, application logs, and the health endpoint provide complementary diagnostic evidence.

## Workshop references

- [5.5 Backend Deployment and Database Integration](../../5-workshop/5.5-backend-and-database/)
- [5.12 Project Handover](../../5-workshop/5.12-project-handover/)
