---
title: "Week 5 - Telemetry and Command API Development"
date: "2026-06-29"
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

> **Period:** 29 June–5 July 2026
> **Role:** Coordinated the backend API contract with the hardware and infrastructure requirements.

## Objectives

- Complete the telemetry ingestion and retrieval APIs.
- Build command creation, polling, and acknowledgement APIs.
- Verify persistent data in PostgreSQL.

## Work completed

| Period | Activity | Recorded outcome |
| :--- | :--- | :--- |
| 29–30 June | Matched the Pydantic telemetry schema to the firmware's camelCase payload | Accepted `deviceId` and `lightIntensity` and mapped them to the data model |
| 1 July | Completed `POST /api/telemetry` | A valid `room_01` payload created a `telemetry_logs` record |
| 2 July | Completed latest and history endpoints | Enabled the dashboard to request `/latest` and `/history` by `device_id` |
| 3–4 July | Completed command creation, pending-command polling, and ACK endpoints | Stored new commands as `Pending` and changed the matching ID to `Executed` after ACK |
| 5 July | Reviewed OpenAPI, ran controlled `curl` requests, and compared SQL records | Matched routes, JSON responses, and database state |

## Weekly outcomes

- Completed telemetry, latest, history, command polling, and ACK flows.
- Verified the command lifecycle by following one command ID through the API and PostgreSQL.
- Documented current limitations: command values were not strictly enum-validated, and ACK ownership was not checked against the route device.

## Challenges and lessons learned

`Pending` can be short-lived when the device polls frequently. Reliable evidence must preserve the command-creation response and then query the same ID after acknowledgement.

## Workshop references

- [5.3 API Specification and Data Flow](../../5-workshop/5.3-architecture-and-service-design/)
- [5.5 Backend and Database](../../5-workshop/5.5-backend-and-database/)
- [5.8 End-to-End Testing and Validation](../../5-workshop/5.8-end-to-end-testing/)
