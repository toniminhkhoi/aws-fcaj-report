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

| Period | Activity | Recorded outcome |
| :--- | :--- | :--- |
| 13 July | Prepared the React, Vite, TypeScript, and Tailwind CSS project and installed dependencies | Ran the dashboard locally |
| 14 July | Configured the Vite proxy for relative `/api` requests | Centralized the EC2 target instead of repeating it across components |
| 15 July | Connected latest, history, and backend health endpoints | Powered telemetry cards, charts, and connection status with API data |
| 16–17 July | Implemented fan, light, curtain, and Auto/Manual controls | Sent valid commands and displayed the command ID and server-side state |
| 18 July | Reviewed real/simulated data handling and light labels | Distinguished simulated data and avoided presenting raw ADC values as Lux |
| 19 July | Used DevTools Network to inspect routes, payloads, responses, and failures | Produced checks for duplicate requests, `Pending` state, and backend errors |

## Weekly outcomes

- Displayed latest and historical `room_01` data.
- Created commands that could be traced through the backend.
- Documented remaining limitations: the UI mode was local state, and simulated data could not be used as operational evidence.

## Challenges and lessons learned

A successful HTTP response proves that the backend accepted a command, not that physical hardware executed it. The UI must wait for ACK/`Executed` or clearly show a pending state.

## Workshop references

- [5.7 Frontend Integration](../../5-workshop/5.7-frontend-integration/)
- [5.8 End-to-End Testing](../../5-workshop/5.8-end-to-end-testing/)
