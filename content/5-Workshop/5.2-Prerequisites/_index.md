---
title: "Prerequisites"
date: "2026-07-28"
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

## Objectives

Prepare the AWS account, local tools, hardware, access material, and baseline knowledge before creating billable resources. The examples use Asia Pacific (Singapore), `ap-southeast-1`, as the workshop baseline; CloudFront/WAF are global services, while S3, ALB, ASG, EC2, RDS, VPC, and CloudWatch resources must match the documented regional design.

## Step 1 - Verify AWS access

- An AWS account with billing visibility and MFA enabled.
- Least-privilege access to S3, CloudFront, WAF, Elastic Load Balancing, Auto Scaling, EC2/EBS, RDS, VPC/Security Groups, IAM Role attachment, CloudWatch, and alarms.
- Permission to pass the approved EC2 IAM Role. AdministratorAccess is not a workshop requirement.
- An EC2 key pair stored locally; never commit `.pem` or `.key` files.
- A known administrator public IP for the SSH rule: `<ADMIN_IP>/32`.

Before provisioning, record the region, VPC CIDR, two application-subnet CIDRs, DB subnets across the required Availability Zones, resource-name prefix, and the person responsible for clean-up.

## Step 2 - Verify local tools and versions

Run each command in the named environment. Compatible newer versions are acceptable when the project source supports them.

| Tool | Environment and check | Expected output |
| :--- | :--- | :--- |
| Git | PowerShell: `git --version` | A Git version |
| Python | PowerShell: `python --version` | A Python version compatible with FastAPI `0.128.8` and SQLAlchemy `2.0.46` |
| Node.js | PowerShell: `node --version` | A version compatible with Vite `8.1.1` and TypeScript `6.0.2` |
| npm | PowerShell: `npm --version` | A numeric npm version |
| PlatformIO | PlatformIO terminal: `pio --version` | PlatformIO Core version |
| PostgreSQL client | PowerShell: `psql --version` | PostgreSQL client version |
| Browser | Open DevTools → Network | Requests can be inspected |
| VS Code | Help → About | Installed version is displayed |

If PowerShell cannot find a tool, reopen the terminal after installation and verify `Get-Command <tool>`. Do not mix `%USERPROFILE%` from Command Prompt with `$env:USERPROFILE` in PowerShell.

## Step 3 - Prepare hardware and electrical safety

Prepare:

- YOLO UNO / ESP32-S3 and a USB data cable;
- DHT20 temperature and humidity sensor;
- analog light sensor;
- fan module and a suitable driver when required;
- light or relay module;
- servo motor for the curtain;
- LCD1602 I2C display used by the final firmware;
- jumper wires, breadboard, and a correctly rated power supply.

Do not power a motor or relay load directly from a microcontroller GPIO. Verify voltage levels, driver/flyback protection, and common ground before connecting actuators.

## Step 4 - Verify source and secrets readiness

The application source is the separate `aws-iot-dashboard` repository. It contains backend `requirements.txt`, frontend `package.json`, firmware `platformio.ini`, `boards/yolo_uno.json`, and `include/secrets.example.h`. Its `.gitignore` excludes `.env`, `.pem`, virtual environments, build output, and `node_modules`; confirm `hardware/include/secrets.h` remains untracked before sharing.

The reviewed source confirms Uvicorn `main:app`, Amazon Linux user `ec2-user`, virtual environment `venv`, tables `devices`, `telemetry_logs`, and `commands`, and the numeric GPIO map documented in section 5.6.

## Step 5 - Confirm prerequisite knowledge

Participants should understand REST methods/status codes, PostgreSQL queries, Security Group references, FastAPI/OpenAPI, React/Vite, PlatformIO, basic Linux permissions, and `systemd`.

## Step 6 - Confirm optional tooling boundaries

| Tool | Required for this implementation? | Reason |
| :--- | :---: | :--- |
| AWS Management Console | Yes | The documented provisioning and evidence flow uses the Console |
| AWS CLI | No | Useful for inventory/verification, but no required deployment command depends on it |
| AWS SAM CLI | No | The project does not deploy Lambda or a SAM application |
| AWS CDK | No | The current infrastructure is not defined as a CDK application |
| Terraform | No | No Terraform configuration/state is part of the reviewed source |
| CloudFormation | No | No CloudFormation stack is created by this Workshop |

Participants may add an approved Infrastructure as Code workflow later, but its resources and clean-up procedure must then be documented separately. Do not block the current Workshop because AWS CLI, SAM, CDK, or Terraform is absent.

## Step 7 - Complete the readiness gate

- [ ] Region and naming are agreed.
- [ ] Least-privilege AWS access works.
- [ ] `<ADMIN_IP>` and EC2 key location are known.
- [ ] All version checks complete.
- [ ] Hardware power requirements are reviewed.
- [ ] Source repositories and ignored secret templates are available.
- [ ] A clean-up owner and evidence location are assigned.

![Verified local development tool versions](/images/5-Workshop/5.2-prerequisites/development-tools-versions.png)
*Figure 2. Terminal evidence for the installed Git, Python, Node.js, npm, and PlatformIO versions used by the project.*

## Expected Result

Proceed only when every blocking item above is satisfied. If a required source repository or the exact firmware pin map is missing, record it as a handover blocker; do not invent values or energize the circuit.

## Troubleshooting

| Symptom | Check |
| :--- | :--- |
| AWS action is denied | Signed-in identity, MFA, required service permission, `iam:PassRole`, and correct account |
| Tool command is not found | Installation path, reopened terminal, and `Get-Command <tool>` |
| Repository is incomplete | Correct branch/commit and presence of backend, frontend, hardware, and example secret files |
| Secret appears tracked | Stop, remove it from the change set, rotate exposed values, and verify `.gitignore` |
| Hardware requirement is unclear | Do not energize actuators; verify the active firmware map and rated supply first |

Next: [review the architecture and service boundaries](../5.3-Architecture-and-Service-Design/).
