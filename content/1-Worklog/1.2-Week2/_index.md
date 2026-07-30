---
title: "Week 2 - AWS Architecture and Network Foundation"
date: "2026-06-08"
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---

> **Period:** 8–14 June 2026
> **Role:** Led the AWS, networking, and IAM design and coordinated API and data-flow boundaries with the backend team.

## Objectives

- Select AWS services that matched the source code and Workshop scope.
- Design the VPC, subnets, Security Groups, and IAM Role according to least privilege.
- Define the telemetry, command, ACK, and monitoring flows.

## Work completed

| Workstream | Work completed | Result/Evidence |
| :--- | :--- | :--- |
| Infrastructure selection | Identified the components required for the Smart Room and selected EC2, EBS, RDS, VPC, IAM Role, and CloudWatch | Service list with the responsibility of each component |
| Network design | Designed the VPC, a public subnet for EC2, and a DB Subnet Group from database subnets | Network diagram placing EC2 and RDS within the intended connectivity boundaries |
| Security Groups | Defined rules for administrative SSH, the demo API, and EC2 → RDS traffic on PostgreSQL 5432 | Rule table restricting SSH by administrator IP and RDS by the EC2 Security Group |
| IAM and monitoring | Defined the EC2 IAM Role and permissions required for CloudWatch Agent logs and metrics | Temporary-role access model without hard-coded AWS access keys |
| System-flow review | Mapped API, database, network ports, identity, and monitoring paths | Checklist of source, destination, port, and required evidence for each connection |
## Weekly outcomes

- Completed the AWS architecture and service boundaries.
- Produced a network, Security Group, and IAM plan suitable for the prototype.
- Clearly separated the current operational model from future options such as Auto Scaling, SQS, and event-driven architecture.

## Challenges and lessons learned

An architecture diagram is useful only when every arrow maps to an actual connection or permission. A smaller service set made the solution easier to deploy, test, and explain.

## Workshop references

- [5.3 Architecture and Service Design](../../5-workshop/5.3-architecture-and-service-design/)
- [5.4 AWS Infrastructure Setup](../../5-workshop/5.4-aws-infrastructure-setup/)
