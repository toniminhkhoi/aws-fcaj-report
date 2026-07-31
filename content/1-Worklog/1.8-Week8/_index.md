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
- Establish logs, metrics, and alarms for ALB/ASG/EC2/RDS.
- Complete the security review, bilingual documentation, demo video, and handover material.

## Work completed

| Workstream | Work completed | Result/Evidence |
| :--- | :--- | :--- |
| Test preparation | Recorded source, firmware, AWS Region, and test conditions and confirmed health | Reproducible baseline for comparing results |
| Telemetry validation | Sent YOLO UNO telemetry, checked latest/history, PostgreSQL, and the dashboard, and ran one controlled `curl` request | Consistent `room_01` data; API/database image isolated FastAPI → RDS evidence |
| Command and actuator validation | Created commands, followed the same ID from `Pending` to `Executed`, and tested fan, light, and curtain controls | Command/ACK matrix passed; video captured physical actuator responses |
| AWS availability and edge | Added CloudFront/WAF/private S3, ALB/target group/ASG, encrypted EBS, and RDS Multi-AZ backup evidence | Two Healthy backends, stable browser/device routes, and primary/standby database evidence |
| CloudWatch | Configured CloudWatch Agent, backend logs, `ec2-rds-metrics`, and eight ALB/ASG/EC2/RDS alarms | Centralized logs, operational metrics, and evaluated alarm states with missing-data notes |
| Operational review | Reviewed the IAM Role, Security Groups, RDS public access, secrets, cost, and current limitations | Security/cost checklist and items for continued improvement |
| Documentation and handover | Reviewed bilingual Workshop pages, READMEs, captions, test matrix, images, video, and checklists | Final package aligned with the deployed system and collected evidence |
## Weekly outcomes

- All telemetry, history, command, polling, actuator, and ACK cases in the test matrix passed.
- Clearly separated API/database evidence from physical hardware evidence.
- CloudWatch received backend logs, displayed ALB/ASG/EC2/RDS metrics, and evaluated eight alarms.
- Completed the bilingual report, Workshop, READMEs, operating guidance, and handover material.
- The demo video shows dashboard interaction and physical device response: [View the demo video](https://drive.google.com/file/d/1T97dUY58hbT2ppxvg7ESR12Jg9BA828W/view?usp=sharing).

## Challenges, resolution, and lessons learned

The `Pending` state was short-lived when the device polled quickly, so the team followed the same command ID before and after ACK. For CloudWatch, `Insufficient data` required checking the agent, namespace, dimensions, and IAM instead of treating it as normal. I learned that every Pass result needs aligned criteria, actual behavior, and evidence, and that documentation must distinguish implemented work from future options.

## Current limitations

CloudFront provides viewer HTTPS, but the ALB/device path uses HTTP; API authentication, WAF blocking, notification actions, and a controlled failover drill remain incomplete. These limitations were recorded for continued improvement.

## Workshop references

- [End-to-End Testing](../../5-workshop/5.8-end-to-end-testing/)
- [CloudWatch Monitoring](../../5-workshop/5.9-cloudwatch-monitoring/)
- [Cost, Security, and Clean-up](../../5-workshop/5.10-cost-security-cleanup/)
- [Results and Future Improvements](../../5-workshop/5.11-results-challenges-future/)
- [Project Handover](../../5-workshop/5.12-project-handover/)
