---
title: "Week 3 - Amazon EC2 and Amazon RDS Deployment"
date: "2026-06-15"
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---

> **Period:** 15–21 June 2026
> **Role:** Provisioned and verified the AWS infrastructure.

## Objectives

- Create the networking and AWS resources required by the backend.
- Launch EC2 with EBS storage and the approved IAM Role.
- Create a private RDS for PostgreSQL database and verify EC2 connectivity.

## Work completed

| Period | Activity | Recorded outcome |
| :--- | :--- | :--- |
| 15 June | Selected `ap-southeast-1` and reviewed the VPC, routes, and DB Subnet Group | Kept EC2 and RDS in the same Region and intended network boundaries |
| 16 June | Created `iot-backend-sg`, `ec2-rds-1`, and `rds-ec2-1` | Used port 8000 for the demo API and restricted PostgreSQL 5432 to EC2-to-RDS traffic |
| 17–18 June | Launched the `t3.micro` `iot-backend-server` with a 10 GiB `gp3` root volume and `iot-dashboard-cloudwatch-role` | EC2 reached `Running` and passed its status checks |
| 19–20 June | Created the `db.t4g.micro` `iot-dashboard-db` RDS for PostgreSQL instance | RDS reached `Available`, used the DB Subnet Group, and had Internet access gateway disabled |
| 21 June | Tested RDS DNS resolution and TCP port 5432 from EC2 | Verified the required route and Security Group path |

## Weekly outcomes

- Prepared EC2, EBS, RDS, IAM, and Security Groups for application deployment.
- Kept RDS private and did not expose PostgreSQL to `0.0.0.0/0`.
- Collected sanitized evidence for EC2, RDS, the IAM Role, and Security Group rules.

## Challenges and lessons learned

Connectivity failures must be separated into network and authentication failures. A successful TCP test confirms routing and Security Groups, but not valid PostgreSQL credentials.

## Workshop references

- [5.4 AWS Infrastructure Setup](../../5-workshop/5.4-aws-infrastructure-setup/)
- [5.10 Cost, Security, and Cleanup](../../5-workshop/5.10-cost-security-cleanup/)
