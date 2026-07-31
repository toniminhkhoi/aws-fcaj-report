---
title: "Cost, Security and Clean-up"
date: "2026-07-28"
weight: 10
chapter: false
pre: " <b> 5.10. </b> "
---

## Overview and objectives

Understand the cost drivers, review the prototype security boundary, preserve required evidence, and remove resources in dependency order. Exact prices are not stated because they depend on region, resource sizes, retention, transfer, and current pricing.

## Step 1 - Review cost drivers

| Resource | Cost driver | Workshop control |
| :--- | :--- | :--- |
| Amazon S3 and CloudFront | Stored objects, requests, transfer, distribution plan | Keep only build artifacts; invalidate deliberately; review plan limits |
| AWS WAF | Web ACL/rule usage and logging | Three managed rule groups currently in Count/Monitor mode; review before enabling paid features |
| Application Load Balancer | Load balancer hours and capacity units | Remove after the workshop if no longer required |
| Amazon EC2 / Auto Scaling | Instance type, desired capacity, and running hours | ASG currently maintains two `t3.micro` instances; scale down/delete only after approval |
| Amazon EBS | Provisioned type/capacity and snapshots | Right-size and remove unattached volumes/snapshots |
| Amazon RDS for PostgreSQL | Multi-AZ DB class, running time, storage, backups | Preserve required snapshots, then delete after evidence if approved |
| Amazon CloudWatch | Custom metrics, log ingestion/storage, alarms | 60-second guest metrics where needed; short retention |
| Data transfer | CloudFront, ALB, device telemetry, and inter-AZ traffic | Reasonable telemetry/polling intervals and cache only static content |

Use the AWS Pricing Calculator or the actual bill for a dated estimate. Do not copy an unverified price into the report.

## Step 2 - Review the security boundary

- Use least-privilege identities and MFA.
- Use an EC2 IAM Role; do not hard-code AWS access keys.
- Restrict SSH port 22 to `<ADMIN_IP>/32`.
- Keep the S3 frontend bucket private with Block Public Access and CloudFront OAC.
- Use viewer HTTPS through CloudFront; keep the three WAF managed rule groups in documented Count/Monitor mode until blocking is tested.
- Allow public HTTP only to the ALB listener; allow backend port 8000 only from the ALB Security Group.
- Allow RDS 5432 only from the EC2 Security Group.
- Keep RDS private.
- Ignore `.env`, `.pem`, `.key`, and `hardware/include/secrets.h`.
- Use placeholders in documentation and redact screenshots.
- Treat ALB HTTP without origin TLS and the unauthenticated API as current limitations.
- Review outbound rules, log retention, database users, and resource tags.
- Rotate any secret immediately if it appears in Git, terminal history, a screenshot, or a demo video.

Current controls include CloudFront viewer HTTPS, WAF monitoring, private S3/OAC, Security Group chaining, encrypted ASG EBS volumes, RDS Multi-AZ, seven-day backups, and a manual snapshot. Production improvements still include ALB HTTPS/origin TLS, authentication, authorization, managed secrets, tested WAF blocking, notification actions, and a reviewed network design.

## Step 3 - Preserve pre-clean-up evidence

Before deleting anything, preserve:

1. architecture and resource inventory;
2. EC2/service health and deployment commit;
3. RDS table/query evidence without credentials;
4. telemetry, command, and ACK test records;
5. CloudWatch log/metric/alarm screenshots; and
6. hardware/frontend demonstration evidence.

Confirm who owns snapshots and how long evidence must be retained.

## AWS Resource Inventory Before Clean-up

The following table summarizes the resources currently used by the project. Required screenshots, logs, validation results, source code, and demonstration videos must be preserved before any resource is stopped or deleted.

| Resource | Name or Role | Current Status | Evidence | Clean-up Action |
| :--- | :--- | :--- | :--- | :--- |
| Amazon S3 | Private frontend bucket | Block Public Access enabled; objects read through CloudFront OAC | S3 policy/OAC evidence | Empty and delete only after the CloudFront origin/distribution is no longer required |
| Amazon CloudFront | `iot-dashboard-frontend` | Active; default S3 behavior and `/api/*` ALB behavior | Origins/Behaviors evidence | Disable, wait for deployment, then delete after removing dependencies |
| AWS WAF | CloudFront-associated web ACL | Three managed rule groups in Count/Monitor mode | WAF rule evidence | Disassociate/delete only with the CloudFront clean-up decision |
| Application Load Balancer | `iot-backend-alb` | Active, listener HTTP:80 | ALB overview/listener | Delete after CloudFront no longer uses the ALB origin and the ASG is detached |
| Target Group | `iot-backend-tg` | Two Healthy targets on HTTP:8000 | Target group evidence | Detach from ALB/ASG, then delete |
| Auto Scaling Group | `iot-backend-asg` | Desired 2, limits 2–4; two Healthy/InService instances | ASG capacity evidence | Set desired/min appropriately only during approved shutdown, then delete the ASG |
| Launch Template and AMI | `iot-backend-template`, `iot-backend-ami-v1` | Launch template version 1 and private AMI available | Launch template/AMI evidence | Delete template versions and deregister the AMI after no ASG/instance depends on them; review AMI snapshots |
| Amazon EC2 | Two ASG backend instances | Running `t3.micro` instances in two Availability Zones | ASG/EC2 pages | Let ASG terminate them during approved ASG clean-up; do not delete one and leave desired capacity unchanged |
| Amazon EBS | Root volumes of ASG instances | `gp3`, 10 GiB, encrypted with `aws/ebs` | EC2 → Volumes page | Verify `Delete on termination`; remove only project-owned retained volumes/snapshots |
| Amazon RDS for PostgreSQL | `iot-dashboard-db` | Available, PostgreSQL, Multi-AZ; primary 1c/standby 1b; seven-day backups | RDS and AWS CLI evidence | Preserve approved snapshots/retained backups, then delete the DB instance |
| RDS snapshot | `iot-dashboard-before-ha-20260730` | Manual snapshot Available | RDS Snapshots | Retain or delete according to the approved data-retention decision |
| DB Subnet Group | `rds-ec2-db-subnet-group-1` | In use by RDS | RDS Connectivity section | Delete only after the DB instance and dependent resources have been removed |
| ALB/EC2 Security Groups | ALB SG, `iot-backend-sg`, `ec2-rds-1` | In use; enforcing ALB → backend → RDS path | Security Group chain | Delete after ALB, EC2, and related ENIs no longer use them |
| RDS Security Group | `rds-ec2-1` | In use; allowing access from the EC2 Security Group | RDS Security Group rules | Delete after RDS and dependent network interfaces have been removed |
| IAM Role | `iot-dashboard-cloudwatch-role` | Attached to EC2 | EC2 Security and IAM Role pages | Detach it from EC2, verify dependencies, and then delete the role |
| IAM Policy | `CloudWatchAgentServerPolicy` | AWS-managed policy attached to the IAM Role | IAM Permissions | Do not delete the AWS-managed policy; remove or delete only the project IAM Role |
| CloudWatch Log Group | `/aws/ec2/aws-iot-dashboard/backend` | Active, containing FastAPI access logs and HTTP status codes | CloudWatch Logs | Preserve required logs, then configure retention or delete the log group |
| CloudWatch Dashboard | `ec2-rds-metrics` | Active, displaying ALB, ASG, EC2, and RDS metrics | CloudWatch Dashboard | Preserve metric evidence, then delete the dashboard when no longer required |
| CloudWatch Alarms | Eight ALB, ASG, EC2, and RDS alarms | Some `OK`, some `Insufficient data`; no notification actions attached | CloudWatch Alarms | Preserve configuration evidence, then delete all eight alarms |
| FastAPI backend service | `aws-iot-backend` on ASG instances | Serving REST APIs and producing access logs | EC2 and CloudWatch Logs | Back up source/service/environment templates before deleting the ASG/AMI |
| Firmware artifact | `firmware.bin` | Successfully built with PlatformIO using the `yolo_uno` environment | Terminal showing `SUCCESS` | Preserve locally or as a repository artifact; it is not an AWS resource to delete |

> **Note:** Do not begin clean-up until all required screenshots, logs, validation results, source code, and demonstration videos have been preserved. The ASG instance volumes shown in the current evidence are encrypted with the AWS managed `aws/ebs` key.

<!-- TODO: Capture this rendered table as aws-resource-inventory.png -->

## Step 4 - Clean up only project-owned resources

1. Preserve screenshots, logs, source code, test results, and demonstration videos; stop new telemetry/command traffic and place actuators in a safe state.
2. Back up frontend/backend source, service files, environment templates, AMI/Launch Template metadata, and the firmware artifact.
3. Decide whether retention requires an approved final RDS snapshot and create it if needed.
4. Disable and delete the CloudFront distribution after deployment completes; disassociate/delete the project WAF web ACL with the same decision.
5. Empty and delete the private S3 frontend bucket only after CloudFront no longer depends on it.
6. Delete the ALB listener/load balancer and detach/delete the target group after CloudFront no longer uses the ALB origin.
7. Delete `iot-backend-asg`; verify its EC2 instances and encrypted root volumes terminate as expected.
8. Delete the Launch Template, deregister the backend AMI, and remove only project-owned AMI snapshots that are no longer required.
9. Delete `iot-dashboard-db` when the database and retained data are no longer required; keep or delete manual/automated snapshots according to the retention decision.
10. Delete all eight project CloudWatch alarms, the `ec2-rds-metrics` dashboard, and configure retention for or delete the backend log groups after preserving evidence.
11. Detach and delete `iot-dashboard-cloudwatch-role` and its instance profile only if no other workload uses them. Do not delete the AWS-managed `CloudWatchAgentServerPolicy`.
12. Delete ALB/backend/RDS Security Groups only after their ALB, EC2, RDS, ENI, and Security Group dependencies are gone.
13. Delete `rds-ec2-db-subnet-group-1` only after RDS no longer uses it. Do not delete selected or shared VPC resources.
14. Recheck CloudFront, WAF, S3, ALB, target groups, ASG, EC2, AMI snapshots, EBS, RDS, CloudWatch, Billing/Cost Explorer, and tagged resources.

The repository does not contain a CloudFormation, SAM, CDK, or Terraform stack/state for this deployment, so the resources above must be inventoried and removed in dependency order rather than by deleting a stack.

Stopping RDS is temporary and subject to service limits; it may start automatically again. Deleting the database and other billable resources is the way to avoid continuing long-term charges, subject to the team's retention decision.

## Step 5 - Verify the clean-up

- Re-run the tagged-resource inventory in the correct region.
- Check for an active CloudFront distribution/WAF, non-empty S3 bucket, ALB/target group, ASG instances, AMI snapshots, unattached EBS volumes, retained RDS snapshots, log groups, and idle alarms.
- If a Security Group cannot be deleted, find the dependent ENI/resource rather than forcing deletion.
- If a database cannot be deleted, review deletion protection and snapshot requirements with the owner.
- Record each deleted resource ID in the clean-up evidence; never expose credentials.

## Expected Result

Required evidence is retained, no unapproved billable project-owned Workshop resource remains, shared resources are untouched, and the security review records current limitations without claiming production readiness.

## Troubleshooting

| Symptom | Check |
| :--- | :--- |
| Security Group cannot be deleted | Find dependent ENIs, EC2, RDS, or referenced Security Groups |
| VPC/subnet cannot be deleted | Check Internet Gateway, route table associations, DB Subnet Group, and ENIs |
| RDS deletion is blocked | Deletion protection, final snapshot name, retained automated backups, and owner approval |
| EBS cost remains | Unattached volumes and snapshots actually owned by this project |
| CloudWatch cost remains | Log-group retention/ingestion and alarms; confirm the agent stopped with EC2 |
| Resource ownership is uncertain | Stop deletion, use tags/inventory and obtain owner confirmation |

Next: [document results, challenges, and future improvements](../5.11-Results-Challenges-Future/).
