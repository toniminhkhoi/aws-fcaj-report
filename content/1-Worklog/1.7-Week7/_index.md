---
title: "Week 7 - Frontend Dashboard Development"
date: "2026-07-13"
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---

> **Period:** 13–19 July 2026
> **Role:** Supported frontend-to-EC2 connectivity and collaborated on dashboard and control-flow validation.

## Objectives

- Display latest telemetry, history, and backend status.
- Provide controls for the fan, light, and curtain.
- Submit commands from the dashboard, inspect backend responses, and identify the missing ACK-tracking behavior.

## Work completed

| Workstream | Work completed | Result/Evidence |
| :--- | :--- | :--- |
| Frontend setup | Prepared React, Vite, TypeScript, Tailwind CSS, and required dependencies | Dashboard ran reliably in the local environment |
| API connectivity | Used relative `/api` paths; the Vite proxy targeted the ALB during development, while the CloudFront `/api/*` behavior handled production routing | Components did not hard-code the backend URL, and development and production paths were documented separately |
| Telemetry display | Connected health, latest, and history and built telemetry cards and charts | Displayed latest data, history, and connection status |
| Device controls | Built fan, light, curtain, and Auto/Manual controls and inspected command-creation requests | UI submitted commands to the backend but did not retain the command ID or track `Pending`/`Executed` state |
| Data and label review | Reviewed the simulated fallback and the description of light values | Identified that fallback data was not explicitly labelled and raw light values were still presented as Lux; recorded both for correction |
| Integration and debugging | Used DevTools Network to inspect routes, payloads, responses, duplicate requests, and API-failure behavior | Verified the production route and found that fallback behavior could report a false success when the backend did not respond |
## Weekly outcomes

- Displayed latest and historical `room_01` data.
- Created commands while the API was available; command IDs and ACK state had to be verified through DevTools, the API, or PostgreSQL rather than the UI itself.
- Documented remaining limitations: UI mode was local state, fallback data was not labelled, and a failed command request could still appear successful.

## Challenges and lessons learned

A successful HTTP response proves that the backend accepted a command, not that physical hardware executed it. The UI must wait for ACK/`Executed` or clearly show a pending state.

## Workshop references

- [5.7 Frontend Integration](../../5-workshop/5.7-frontend-integration/)
- [5.8 End-to-End Testing](../../5-workshop/5.8-end-to-end-testing/)
