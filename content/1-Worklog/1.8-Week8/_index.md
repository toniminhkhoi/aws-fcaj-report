---
title: "Week 8 - Integration, Testing, CloudWatch, and Handover"
date: "2026-07-20"
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---

> **Period:** 20–31 July 2026
>
> **Role:** Coordinated full-system integration and took responsibility for AWS/hardware troubleshooting, CloudWatch, and technical-evidence review.

## Objectives

- Test the complete telemetry and command flows with physical hardware.
- Trace the same data or command ID across the dashboard, API, RDS, and firmware.
- Establish logs, metrics, and alarms for EC2/RDS.
- Complete the security review, bilingual documentation, demo video, and handover material.

## Work completed

| Period | Activity | Recorded result |
| :--- | :--- | :--- |
| 20 Jul | Recorded source, firmware, AWS Region, and test conditions | Established a reproducible configuration baseline |
| 21 Jul | Checked health, POST telemetry, latest data, and history | The API and PostgreSQL returned consistent `room_01` data |
| 22 Jul | Created a command, observed `Pending`, sent ACK, and queried the same ID | The same row changed to `Executed` |
| 23–24 Jul | Tested `FAN_*`, `LIGHT_*`, and `CURTAIN_*` from the dashboard | The demo video captured the fan, light, and servo responses |
| 25 Jul | Sent a controlled payload with `curl` and compared it with RDS | Isolated FastAPI → RDS validation without presenting it as hardware evidence |
| 26–27 Jul | Configured CloudWatch Agent and `/aws/ec2/aws-iot-dashboard/backend` | Centralized backend logs |
| 28 Jul | Built `ec2-rds-metrics` for EC2 CPU/disk and RDS CPU/connections | Created an operational view of deployed resources |
| 29 Jul | Checked five EC2/RDS alarms and investigated `Insufficient data` | Documented the agent, namespace, dimension, and IAM dependencies for memory/disk |
| 30 Jul | Reviewed IAM Role, Security Groups, RDS public access, secrets, and cost | Recorded current controls and HTTPS, authentication, and High Availability gaps |
| 31 Jul | Reviewed bilingual Workshop pages, READMEs, captions, test matrix, video, and handover checklist | Aligned the final material with the `room_01` prototype and collected evidence |

## Weekly outcomes

- All telemetry, history, command, polling, actuator, and ACK cases in the test matrix passed.
- Clearly separated API/database evidence from physical hardware evidence.
- CloudWatch received backend logs, displayed EC2/RDS metrics, and evaluated five alarms.
- Completed the bilingual report, Workshop, READMEs, operating guidance, and handover material.
- The demo video shows dashboard interaction and physical device response: [View the demo video](https://drive.google.com/file/d/1T97dUY58hbT2ppxvg7ESR12Jg9BA828W/view?usp=sharing).

## Challenges, resolution, and lessons learned

The `Pending` state was short-lived when the device polled quickly, so the team followed the same command ID before and after ACK. For CloudWatch, `Insufficient data` required checking the agent, namespace, dimensions, and IAM instead of treating it as normal. I learned that every Pass result needs aligned criteria, actual behavior, and evidence, and that documentation must distinguish implemented work from future options.

## Limitations and future work

The current model uses HTTP without TLS, has no API authentication, and runs on one EC2 instance with one RDS database. HTTPS, user/device authentication, AWS IoT Core, queueing, High Availability, and multi-device scaling remain future options and were not deployed.

## Workshop references

- [End-to-End Testing](../../5-workshop/5.8-end-to-end-testing/)
- [CloudWatch Monitoring](../../5-workshop/5.9-cloudwatch-monitoring/)
- [Cost, Security, and Clean-up](../../5-workshop/5.10-cost-security-cleanup/)
- [Results and Future Improvements](../../5-workshop/5.11-results-challenges-future/)
- [Project Handover](../../5-workshop/5.12-project-handover/)
