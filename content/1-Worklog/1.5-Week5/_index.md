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

| Workstream | Work completed | Result/Evidence |
| :--- | :--- | :--- |
| Data contract | Matched the Pydantic schema to the firmware camelCase payload and mapped `deviceId` and `lightIntensity` to the data model | Consistent contract across firmware, FastAPI, and PostgreSQL |
| Telemetry ingestion | Completed and tested `POST /api/telemetry` with `room_01` as the `device_id` value | Valid telemetry created a new `telemetry_logs` row |
| Latest and history | Completed latest-data and history APIs by `device_id` | Supplied data for dashboard telemetry cards and charts |
| Command lifecycle | Completed command creation, pending-command polling, and ACK APIs | Stored commands as `Pending` and changed the same ID to `Executed` after ACK |
| API/database validation | Reviewed OpenAPI, sent controlled `curl` requests, and compared SQL records | Routes, JSON responses, and database state matched |
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
