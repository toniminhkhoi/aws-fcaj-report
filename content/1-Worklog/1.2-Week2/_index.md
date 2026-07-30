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

| Period | Activity | Recorded outcome |
| :--- | :--- | :--- |
| 8–9 June | Compared EC2/RDS with managed serverless and IoT alternatives | Selected EC2, EBS, RDS, VPC, IAM Role, and CloudWatch; documented that Lambda, API Gateway, DynamoDB, S3, and AWS IoT Core were not implemented |
| 10 June | Designed a VPC with a public subnet for EC2 and private database subnets | Kept RDS private and allowed PostgreSQL connectivity from EC2 only |
| 11 June | Designed Security Group rules for administrative SSH, the demo API, and EC2-to-RDS traffic | Restricted SSH by administrator IP and scoped RDS port 5432 to the EC2 Security Group |
| 12 June | Defined the EC2 IAM Role required by CloudWatch Agent | Used temporary role credentials instead of hard-coded AWS access keys |
| 13–14 June | Mapped the architecture to API calls, database operations, and metric/log paths | Assigned a source, destination, port, identity, and evidence requirement to each connection |

## Weekly outcomes

- Completed the AWS architecture and service boundaries.
- Produced a network, Security Group, and IAM plan suitable for the prototype.
- Clearly separated the current operational model from future options such as Auto Scaling, SQS, and event-driven architecture.

## Challenges and lessons learned

An architecture diagram is useful only when every arrow maps to an actual connection or permission. A smaller service set made the solution easier to deploy, test, and explain.

## Workshop references

- [5.3 Architecture and Service Design](../../5-workshop/5.3-architecture-and-service-design/)
- [5.4 AWS Infrastructure Setup](../../5-workshop/5.4-aws-infrastructure-setup/)
