---
title: "Week 1 - Requirements Analysis and Planning"
date: "2026-06-01"
weight: 1
chapter: false
pre: " <b> 1.1. </b> "
---

> **Period:** 1–7 June 2026
> **Role:** Contributed to requirements analysis and represented the AWS and hardware perspective in the team plan.

## Objectives

- Define the problem, target users, and scope of the prototype.
- Agree on deliverables, measurable success criteria, and team responsibilities.
- Draft the initial telemetry and command architecture.

## Work completed

| Period | Activity | Recorded outcome |
| :--- | :--- | :--- |
| 1–2 June | Analyzed the need to monitor temperature, humidity, and light and to control a fan, light, and curtain for a sample room | Limited acceptance scope to `room_01`; the project was not presented as a multi-site BMS |
| 3 June | Identified AWS learners, room operators, maintainers, and FCAJ reviewers as the primary users | Documented the need and expected value for each user group |
| 4–5 June | Defined functional requirements for telemetry, latest data, history, commands, and ACK | Produced an observable output for each function |
| 6 June | Agreed on AWS/Hardware, Backend, Frontend/Integration, and Documentation/QA roles | Established an ownership matrix for contribution traceability |
| 7 June | Drafted an architecture with YOLO UNO, FastAPI on EC2, RDS PostgreSQL, a React dashboard, and CloudWatch | Produced the initial architecture and risk list |

## Weekly outcomes

- Completed the initial scope, ownership model, and system architecture.
- Defined measurable criteria including an HTTP 200 health check, persisted telemetry, `Pending` → `Executed` command state, and observable CloudWatch data.
- Agreed to claim only components supported by source code or deployment evidence.

## Challenges and lessons learned

The initial scope could easily be described as larger than a one-room prototype. The main lesson was to separate learning objectives, implemented behavior, and future scaling options.

## Workshop references

- [5.1 Workshop Overview](../../5-workshop/5.1-workshop-overview/)
- [5.11 Results, Challenges, and Future Improvements](../../5-workshop/5.11-results-challenges-future/)
