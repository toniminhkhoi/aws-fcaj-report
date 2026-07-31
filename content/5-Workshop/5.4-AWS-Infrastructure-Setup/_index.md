---
title: "AWS Infrastructure Setup"
date: "2026-07-28"
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

## Overview and objectives

Create the network, identity, edge, load-balancing, compute, storage, and database foundation for the deployed system. Use placeholders in notes and screenshots; never publish an account ID, password, private endpoint, or key.

## Step 1 - Select the region and address plan

In the AWS Console, select the confirmed project region. This workshop uses **Asia Pacific (Singapore), `ap-southeast-1`**, as its baseline. Record non-overlapping CIDRs for the VPC, one public subnet, and at least two DB subnets in different Availability Zones.

**Expected result:** every resource created below appears in the same region and uses the agreed name prefix.

## Step 2 - Create or select the VPC and subnets

1. Open **VPC → Your VPCs** and create/select the project VPC.
2. Enable DNS resolution and DNS hostnames.
3. Create/select public application subnets in at least `ap-southeast-1a` and `ap-southeast-1c` for the ALB and backend ASG.
4. Attach an Internet Gateway to the VPC.
5. Add `0.0.0.0 → Internet Gateway` to the public application-subnet route table.
6. Create/select database subnets in separate Availability Zones, including the verified primary/standby placement in `ap-southeast-1c` and `ap-southeast-1b`. Do not add an Internet Gateway route for private DB subnets.
7. In **RDS → Subnet groups**, create a DB Subnet Group containing both DB subnets.

**Expected result:** the ALB spans the application subnets, the ASG can place two backend instances across AZs, and the database subnets remain private.

## Step 2A - Configure private S3, CloudFront, and WAF

1. Build the React + Vite frontend and upload the `dist` artifacts to the project S3 bucket.
2. Enable **Block all public access** on the bucket.
3. Create a CloudFront Origin Access Control (OAC) and allow the CloudFront distribution to read only the required S3 objects.
4. Configure two CloudFront origins: the private S3 bucket for static files and `iot-backend-alb` for the API.
5. Keep the default `*` behavior on the S3 origin with managed optimized caching.
6. Create a higher-priority `/api/*` behavior for the ALB origin, disable caching, forward the required viewer values except the `Host` header, and allow the HTTP methods required by the API.
7. Associate the CloudFront-created WAF web ACL. The three managed rule groups currently run in **Count/Monitor mode**, so they observe requests but do not yet block them.

CloudFront provides HTTPS to browser viewers. In the verified configuration, CloudFront reaches the ALB origin over HTTP and YOLO UNO also calls the ALB directly over HTTP; do not describe these two paths as end-to-end TLS.

![CloudFront origins for the private S3 frontend and ALB API](/images/5-Workshop/5.4-aws-infrastructure/cloudfront-distribution-origins.png)
*Figure 3a. The distribution has separate private-S3 and ALB origins.*

![CloudFront behaviors for static content and API requests](/images/5-Workshop/5.4-aws-infrastructure/cloudfront-behaviors.png)
*Figure 3b. The default behavior serves S3 content, while `/api/*` has higher priority and uses the ALB origin.*

![Private S3 bucket protected by Block Public Access and CloudFront OAC](/images/5-Workshop/5.4-aws-infrastructure/s3-private-oac.png)
*Figure 3c. The frontend bucket remains private and grants object reads to the CloudFront distribution through OAC.*

![AWS WAF web ACL with three managed rule groups](/images/5-Workshop/5.4-aws-infrastructure/waf-web-acl-three-rules.png)
*Figure 3d. Three AWS managed rule groups are associated with the distribution in Count mode.*

## Step 3 - Create Security Groups

Create the ALB, backend, and RDS Security Groups in the same VPC. The deployed environment uses a separate ALB Security Group, `iot-backend-sg` for backend targets, and `rds-ec2-1` for the RDS-side rule.

| Security Group | Type | Source | Purpose |
| :--- | :---: | :--- | :--- |
| ALB Security Group | HTTP 80 | `0.0.0.0/0` | Public ALB listener and CloudFront API origin |
| `iot-backend-sg` | Custom TCP 8000 | ALB Security Group | FastAPI target traffic only |
| `iot-backend-sg` | SSH 22 | `<ADMIN_IP>/32` | Restricted administration when required |
| `iot-backend-sg` → `rds-ec2-1` | PostgreSQL 5432 | Backend Security Group reference | Backend-to-RDS only |

In the current environment, port `8000` is not the public application entry point; it accepts target traffic from the ALB Security Group. CloudFront forwards browser `/api/*` requests to the ALB, while YOLO UNO calls the ALB DNS endpoint directly. RDS does not expose PostgreSQL port `5432` to `0.0.0.0/0`; it accepts database connections from the backend Security Group.

The following two screenshots separate the EC2-side rules from the RDS-side Security Group relationship while redacting the administrator IP and other sensitive identifiers.

![ALB, backend, and RDS Security Group chain](/images/5-Workshop/5.4-aws-infrastructure/security-group-chain.png)
*Figure 7a. Security Group chain: public HTTP to the ALB, ALB-to-backend traffic on port 8000, and backend-to-RDS traffic on port 5432.*

![RDS Security Group relationship with the backend Security Group](/images/5-Workshop/5.4-aws-infrastructure/rds-security-group.png)
*Figure 7b. RDS accepts PostgreSQL port 5432 from the backend Security Group rather than a public CIDR.*

## Step 4 - Create the EC2 IAM Role

1. Open **IAM → Roles → Create role**.
2. Select trusted entity **AWS service → EC2**.
3. Attach `CloudWatchAgentServerPolicy` only if CloudWatch Agent will publish metrics/logs.
4. Use the deployed role name `iot-dashboard-cloudwatch-role` and create an instance profile.

Do not create long-lived access keys. The deployed role uses the AWS-managed `CloudWatchAgentServerPolicy`, allowing CloudWatch Agent to publish the required logs and metrics without hard-coded AWS access keys. The role grants permission; CloudWatch Agent is installed separately in section 5.9. Review and narrow IAM permissions for a production deployment instead of assuming the managed policy is perfectly least-privileged.

The EC2 Security page and IAM Role details confirm the role attachment and the AWS-managed policy.

![IAM Role and CloudWatchAgentServerPolicy attached to EC2](/images/5-Workshop/5.4-aws-infrastructure/ec2-iam-role.png)
*Figure 5. The EC2 instance is attached to the `iot-dashboard-cloudwatch-role`, which uses `CloudWatchAgentServerPolicy` to allow CloudWatch Agent to publish logs and metrics without hard-coded AWS access keys.*

## Step 5 - Prepare the AMI, launch template, ASG, and EBS

1. Validate FastAPI, the systemd service, CloudWatch Agent, and the local health check on the source instance.
2. Create the private AMI `iot-backend-ami-v1` only after validation.
3. Create launch template `iot-backend-template` with the AMI, `t3.micro`, backend Security Group, key pair if required, and the CloudWatch IAM instance profile.
4. Configure a 10 GiB `gp3` root EBS volume encrypted with the AWS managed `aws/ebs` KMS key.
5. Create `iot-backend-asg` across the application subnets with minimum/desired/maximum capacity `2/2/4`.
6. Wait for both instances to become **InService** and **Healthy**.

The verified ASG instances use 10 GiB gp3 EBS volumes encrypted with the AWS managed `aws/ebs` KMS key. Keep encryption enabled in every launch-template version.

Do not use an instance public IP as the application endpoint. Record the ALB DNS name privately for firmware/health checks and the CloudFront domain for browser access.

The EC2 Instances page confirms the deployed compute size, Availability Zone, running state, and health checks.

![AMI and launch template](/images/5-Workshop/5.4-aws-infrastructure/launch-template-ami.png)
*Figure 4a. The validated backend AMI is used by the launch template.*

![ASG capacity and instances](/images/5-Workshop/5.4-aws-infrastructure/asg-capacity-instances.png)
*Figure 4b. The ASG maintains two healthy in-service instances with scaling limits from 2 to 4.*

![Encrypted EBS volumes](/images/5-Workshop/5.4-aws-infrastructure/ebs-encryption-kms.png)
*Figure 4c. Backend gp3 EBS volumes are encrypted with the AWS managed `aws/ebs` key.*

## Step 5A - Create the target group and Application Load Balancer

1. Create `iot-backend-tg` as an instance target group on HTTP port `8000` with health path `/api/health`.
2. Create internet-facing `iot-backend-alb` across the two application subnets.
3. Add an HTTP port `80` listener whose default action forwards to `iot-backend-tg`.
4. Attach the ASG to the target group and wait for both registered targets to become **Healthy**.

![Application Load Balancer overview](/images/5-Workshop/5.4-aws-infrastructure/alb-overview.png)
*Figure 5a. The internet-facing ALB is Active across two Availability Zones.*

![ALB listener forwarding](/images/5-Workshop/5.4-aws-infrastructure/alb-listener-forwarding.png)
*Figure 5b. The HTTP:80 listener forwards to `iot-backend-tg`.*

![Healthy target group](/images/5-Workshop/5.4-aws-infrastructure/target-group-healthy.png)
*Figure 5c. Two backend targets are healthy on port 8000.*

## Step 6 - Create Amazon RDS for PostgreSQL

1. Open **RDS → Databases → Create database** and select PostgreSQL.
2. Choose the deployed `db.t4g.micro` class and the approved storage configuration.
3. Set the initial database name to `iot_dashboard`.
4. Select the project VPC, DB Subnet Group `rds-ec2-db-subnet-group-1`, and the deployed RDS Security Groups.
5. Keep Internet access disabled and enable Multi-AZ.
6. Store the master password securely as `<DB_PASSWORD>`; do not add it to screenshots or Git.
7. Set automated backup retention to seven days and keep encryption/monitoring settings approved for the project.
8. Wait for `iot-dashboard-db` to reach **Available**; verify primary `ap-southeast-1c`, standby `ap-southeast-1b`, and record `<RDS_ENDPOINT>` privately.
9. Take a manual snapshot before material HA or application changes.

Multi-AZ is enabled for managed failover through the same RDS endpoint. Do not describe the standby as a read replica or claim RDS Proxy, a public endpoint, or IAM database authentication.

The RDS summary and Connectivity & security page confirm the PostgreSQL engine, instance class, DB Subnet Group, Availability Zone, and disabled Internet access gateway.

![Amazon RDS PostgreSQL instance using a DB Subnet Group](/images/5-Workshop/5.4-aws-infrastructure/rds-postgresql-available.png)
*Figure 6. The `iot-dashboard-db` Amazon RDS for PostgreSQL instance in the Available state, using the `rds-ec2-db-subnet-group-1` DB Subnet Group with the Internet access gateway disabled.*

![RDS primary and standby Availability Zones](/images/5-Workshop/5.4-aws-infrastructure/rds-primary-standby-az.png)
*Figure 6a. RDS Multi-AZ reports the primary in `ap-southeast-1c` and standby in `ap-southeast-1b`.*

![RDS automated backup retention](/images/5-Workshop/5.4-aws-infrastructure/rds-backup-retention.png)
*Figure 6b. Automated backups are retained for seven days.*

![Manual RDS snapshot](/images/5-Workshop/5.4-aws-infrastructure/rds-manual-snapshot.png)
*Figure 6c. A manual snapshot provides a named recovery point.*

## Step 7 - Verify access and network

For restricted administration, connect from Windows PowerShell only when an instance route and approved SSH rule are available:

```powershell
ssh -i "$env:USERPROFILE\.ssh\<KEY_FILE>.pem" <EC2_USER>@<EC2_PUBLIC_IP>
```

From EC2 Linux Bash, verify DNS and TCP reachability:

```bash
getent hosts <RDS_ENDPOINT>
nc -vz <RDS_ENDPOINT> 5432
```

If `nc` is unavailable, install the appropriate netcat package for the selected Linux distribution. A successful TCP test proves the route and Security Groups, not database credentials.

From an external client, verify the public API through the ALB instead of port 8000 on an instance:

```powershell
curl.exe -sS -i "http://<ALB_DNS_NAME>/api/health"
```

## Expected Result and Evidence

The figures above provide redacted evidence of the IAM Role, AMI/launch template, ASG capacity, encrypted EBS, active ALB, two healthy targets, RDS **Available** state, Multi-AZ placement, backups, snapshot, and Security Group chain. Preserve separate command output for the successful backend-to-RDS port test.

## Troubleshooting

| Symptom | Check |
| :--- | :--- |
| SSH timeout | Public IP, route table, Internet Gateway, `<ADMIN_IP>/32`, local firewall |
| SSH permission denied | Login user and local key permissions; never change the remote SG to solve a key error |
| RDS timeout | Endpoint/region, DB subnet routing, RDS SG source, network ACL |
| RDS connection refused | Correct port and database status |
| EC2 cannot publish metrics | IAM instance profile attachment and outbound HTTPS |
| Browser API request fails | CloudFront `/api/*` behavior, ALB listener, target health, backend bind address, and service state |
| Device API timeout | ALB DNS, listener port 80, target health, firmware route, and client network |

Next: [deploy FastAPI and connect PostgreSQL](../5.5-Backend-and-Database/).
