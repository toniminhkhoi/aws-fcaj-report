---
title: "Worklog"
date: "2026-06-01"
weight: 1
chapter: false
pre: " <b> 1. </b> "
---

This worklog records the implementation of the **AWS IoT Monitoring and Control Dashboard** during the internship from **1 June 2026 to 31 July 2026**. It follows the Proposal timeline and the technical evidence documented in the Workshop.

The period covers 8 weeks and 5 days. The worklog is organized into eight phases: the first seven are seven-day weeks, while Week 8 runs from 20 to 31 July and includes the remaining integration, monitoring, documentation, and handover work.

This was a team project. Each weekly page distinguishes my work as the **AWS and Hardware Lead** from tasks completed in collaboration with the backend, frontend, and documentation members.

The log is organized by the planned internship work phases and cross-checked against the final system, AWS screenshots, demo video, and handover documents. The application repository history begins on 26 June 2026 and includes batches of changes, so commit timestamps alone are not treated as proof of the completion date of each activity.

| Week | Period | Implementation milestone | Main outcome |
| :--: | :--- | :----------------------- | :----------- |
| 1 | 1–7 June | [Requirements analysis and planning](1.1-week1/) | Defined the problem, `room_01` scope, responsibilities, and initial architecture |
| 2 | 8–14 June | [AWS architecture and networking foundation](1.2-week2/) | Completed VPC, subnet, Security Group, IAM, and data-flow design |
| 3 | 15–21 June | [Amazon EC2 and Amazon RDS deployment](1.3-week3/) | Started EC2, prepared RDS PostgreSQL, and verified network connectivity |
| 4 | 22–28 June | [Backend and database foundation](1.4-week4/) | Ran FastAPI under `systemd`, connected RDS, and created application tables |
| 5 | 29 June–5 July | [Telemetry and command API development](1.5-week5/) | Completed telemetry, latest, history, command polling, and ACK flows |
| 6 | 6–12 July | [YOLO UNO hardware integration](1.6-week6/) | Read sensors, controlled actuators, connected Wi-Fi, sent telemetry, and processed commands |
| 7 | 13–19 July | [Frontend dashboard development](1.7-week7/) | Displayed telemetry/history, submitted commands, and documented the ACK-tracking gap |
| 8 | 20–31 July | [Integration, testing, CloudWatch, and handover](1.8-week8/) | Completed end-to-end validation, monitoring, security review, documentation, and the demo |
