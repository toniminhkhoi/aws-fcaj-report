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

| Workstream | Work completed | Result/Evidence |
| :--- | :--- | :--- |
| AWS environment preparation | Selected `ap-southeast-1` and reviewed the VPC, routes, subnets, and DB Subnet Group before provisioning | EC2 and RDS placed in the same Region and intended network design |
| EC2 and EBS deployment | Launched the `t3.micro` `iot-backend-server` with 10 GiB gp3 EBS and `iot-dashboard-cloudwatch-role` | EC2 reached `Running`, passed status checks, and had the role attached |
| RDS deployment | Created the `db.t4g.micro` `iot-dashboard-db` RDS for PostgreSQL instance | Database reached `Available`, used the DB Subnet Group, and had no public access |
| Connectivity controls | Created `iot-backend-sg`, `ec2-rds-1`, and `rds-ec2-1` and scoped PostgreSQL 5432 by Security Group | EC2 reached RDS without exposing the database directly to the Internet |
| Verification and evidence | Checked RDS DNS/port 5432 from EC2 and reviewed EC2, EBS, RDS, IAM Role, and network status | EC2 Running, RDS Available, IAM Role, and Security Group screenshots for the Workshop |
## Weekly outcomes

- Prepared EC2, EBS, RDS, IAM, and Security Groups for application deployment.
- Kept RDS private and did not expose PostgreSQL to `0.0.0.0/0`.
- Collected sanitized evidence for EC2, RDS, the IAM Role, and Security Group rules.

## Challenges and lessons learned

Connectivity failures must be separated into network and authentication failures. A successful TCP test confirms routing and Security Groups, but not valid PostgreSQL credentials.

## Workshop references

- [5.4 AWS Infrastructure Setup](../../5-workshop/5.4-aws-infrastructure-setup/)
- [5.10 Cost, Security, and Cleanup](../../5-workshop/5.10-cost-security-cleanup/)
