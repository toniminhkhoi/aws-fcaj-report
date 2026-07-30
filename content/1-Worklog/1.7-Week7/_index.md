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
- Track a command from submission until backend acknowledgement.

## Work completed

| Workstream | Work completed | Result/Evidence |
| :--- | :--- | :--- |
| Frontend setup | Prepared React, Vite, TypeScript, Tailwind CSS, and required dependencies | Dashboard ran reliably in the local environment |
| API connectivity | Used a Vite proxy for relative `/api` requests and centralized the backend URL | Frontend reached EC2 without duplicating the URL across components |
| Telemetry display | Connected health, latest, and history and built telemetry cards and charts | Displayed latest data, history, and connection status |
| Device controls | Built fan, light, curtain, and Auto/Manual controls and displayed command ID/state | UI sent valid commands and exposed backend responses for tracking |
| Data and label review | Distinguished real/simulated data and reviewed the description of light values | Simulated data was identifiable and raw ADC values were not presented as Lux |
| Integration and debugging | Used DevTools Network to inspect routes, payloads, responses, duplicate requests, and `Pending` state | Resolved frontend–backend issues before end-to-end validation |
## Weekly outcomes

- Displayed latest and historical `room_01` data.
- Created commands that could be traced through the backend.
- Documented remaining limitations: the UI mode was local state, and simulated data could not be used as operational evidence.

## Challenges and lessons learned

A successful HTTP response proves that the backend accepted a command, not that physical hardware executed it. The UI must wait for ACK/`Executed` or clearly show a pending state.

## Workshop references

- [5.7 Frontend Integration](../../5-workshop/5.7-frontend-integration/)
- [5.8 End-to-End Testing](../../5-workshop/5.8-end-to-end-testing/)
