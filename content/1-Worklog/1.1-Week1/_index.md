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

| Workstream | Work completed | Result/Evidence |
| :--- | :--- | :--- |
| Requirements analysis | Worked with the team to define temperature, humidity, and light monitoring plus fan, light, and curtain control for the Smart Room | Functional list covering telemetry, latest/history, commands, and ACK |
| Scope definition | Used `room_01` as the `device_id` for the sample room | Scope and acceptance criteria recorded in the Proposal |
| User analysis | Analyzed the needs of AWS learners, room operators, maintainers, and FCAJ reviewers | Target-user list and expected value for each group |
| Architecture design | Drafted YOLO UNO → FastAPI on EC2 → RDS PostgreSQL → React Dashboard with CloudWatch | Initial architecture and two-way data-flow diagram |
| Ownership and planning | Assigned AWS/Hardware, Backend, Frontend/Integration, and Documentation/QA responsibilities and divided delivery into eight weeks | Ownership table, 1 June–31 July timeline, and initial risk list |
## Weekly outcomes

- Completed the initial scope, ownership model, and system architecture.
- Defined measurable criteria including an HTTP 200 health check, persisted telemetry, `Pending` → `Executed` command state, and observable CloudWatch data.
- Agreed to claim only components supported by source code or deployment evidence.

## Challenges and lessons learned

The initial challenge was translating the Smart Room idea into specific requirements and test criteria. The main lesson was that each function needs an observable output so the team can validate it and collect evidence.

## Workshop references

- [5.1 Workshop Overview](../../5-workshop/5.1-workshop-overview/)
- [5.11 Results, Challenges, and Future Improvements](../../5-workshop/5.11-results-challenges-future/)
