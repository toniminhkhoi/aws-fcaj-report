---
title: "Architecture and Service Design"
date: "2026-07-28"
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

## Architecture

![AWS IoT Monitoring and Control Dashboard architecture](/images/2-Proposal/IoT_Dashboard_Architecture.png)

*Figure 5-2. Current architecture: CloudFront/WAF with a private S3 origin, `/api/*` forwarding to ALB, two FastAPI instances in an ASG, RDS PostgreSQL Multi-AZ, and the direct device-to-ALB path.*

The dashboard user and YOLO UNO are outside AWS. CloudFront distributes the React + Vite build from a private S3 bucket and forwards browser `/api/*` requests to the internet-facing ALB. YOLO UNO uses the ALB DNS name directly. Inside the VPC, the ALB routes to two FastAPI instances managed by an ASG in `ap-southeast-1a` and `ap-southeast-1c`; RDS PostgreSQL Multi-AZ uses a primary in `ap-southeast-1c` and standby in `ap-southeast-1b`. Each backend instance has an encrypted EBS root volume.

![Browser, device, command, acknowledgement, and monitoring data flows](/images/5-Workshop/5.3-architecture/architecture-data-flows.svg)

*Figure 5-3. The browser uses CloudFront, while YOLO UNO uses the ALB directly; both routes converge on the same FastAPI/RDS command and telemetry model.*

## Components and AWS service selection

| Component/service | Responsibility and reason |
| :--- | :--- |
| React + Vite + TypeScript + Tailwind CSS | Operator UI built to private S3 and distributed through CloudFront |
| Amazon S3 + CloudFront + AWS WAF | Private static origin, HTTPS edge delivery, `/api/*` routing, and managed-rule monitoring |
| Application Load Balancer + Target Group | Stable API entry point and `/api/health` checks for backend targets |
| EC2 Auto Scaling Group | Maintains two FastAPI/Uvicorn instances and can scale from 2 to 4 |
| Amazon EC2 | Full control of FastAPI, Python, Uvicorn, and `systemd` on each backend instance |
| Amazon EBS | Encrypted persistent root volume attached to each EC2 instance |
| Amazon RDS for PostgreSQL Multi-AZ | Managed relational persistence with primary/standby failover |
| Amazon VPC and subnets | Network boundaries for ALB, ASG instances, and the DB Subnet Group |
| Security Groups | Stateful allow rules for ALB-to-backend and backend-to-RDS traffic |
| AWS IAM Role | Supplies temporary permission for EC2 to publish monitoring data |
| CloudWatch Agent | Software on EC2 that reads guest metrics/log files |
| Amazon CloudWatch/Alarms | Stores metrics/logs and evaluates thresholds |
| YOLO UNO / ESP32-S3 | Reads sensors, controls actuators, polls commands, and sends ACK |

An IAM Role grants permission; it is not the CloudWatch Agent. The agent is a process installed and managed on EC2.

## 5.3.3 AWS Service Selection and Architectural Trade-offs

The project selects AWS services based on four primary criteria:

1. Compatibility with the current architecture and source code.
2. Simplicity for implementation and workshop delivery.
3. Direct observability, testing, and operational control.
4. Reasonable cost for a learning and demonstration environment.

Not every serverless service is required for this use case. The current system runs a persistent FastAPI application, connects to PostgreSQL, and communicates with YOLO UNO hardware through HTTP REST APIs. Therefore, the team selected Amazon EC2 and Amazon RDS instead of redesigning the entire platform around Lambda, API Gateway, and DynamoDB.

### Selected AWS Services

| AWS Service | Role in the System | Selection Reason | Trade-off |
| :--- | :--- | :--- | :--- |
| **Amazon EC2** | Hosts the FastAPI backend, Uvicorn, and CloudWatch Agent | Provides full control over the Python environment, dependencies, networking, systemd service, and log files | The team must manage the operating system, packages, services, and part of the security configuration |
| **Amazon EBS** | Provides the EC2 root volume | Persists the operating system, source code, virtual environment, and local logs | An unattached volume may continue generating charges if it is not deleted |
| **Amazon RDS for PostgreSQL** | Stores telemetry and command states | PostgreSQL is suitable for structured telemetry and command records, while RDS reduces database administration effort | A continuously running database instance may cost more than some serverless options at very low usage |
| **Amazon VPC** | Provides the network environment for EC2 and RDS | Enables subnet, routing, and network-access control | Requires correct network and Security Group configuration |
| **Security Groups** | Control inbound traffic to EC2 and RDS | Restrict SSH to the administrator IP and PostgreSQL access to the EC2 Security Group | Incorrect rules may block the application or expose services unnecessarily |
| **AWS IAM Role** | Grants EC2 permission to publish monitoring data | Avoids storing AWS access keys on the instance or in source code | Policies must follow the principle of least privilege |
| **Amazon CloudWatch** | Collects backend logs and infrastructure metrics | Centralizes operational data for troubleshooting and validation | Log ingestion, retention, and custom metrics may generate additional costs |
| **CloudWatch Alarms** | Evaluates CPU, memory, disk, and database-connection thresholds | Provides visibility when configured operating thresholds are exceeded | Alarm usefulness depends on appropriate thresholds and evaluation periods |
| **Amazon S3 and CloudFront** | Hosts and distributes the frontend and routes `/api/*` to ALB | Keeps the S3 origin private through OAC and provides the browser HTTPS endpoint | Cache/behavior changes require controlled deployment and invalidation |
| **AWS WAF** | Applies three managed rule groups at the CloudFront edge | Provides visibility into common web threats before enabling enforcement | Current rules run in Count/Monitor mode, not blocking mode |
| **Application Load Balancer and Auto Scaling** | Distributes API requests to two healthy FastAPI targets | Removes a single backend endpoint and maintains desired capacity across two AZs | Adds ALB/EC2 running cost and launch-template lifecycle management |

### Why Amazon EC2 Was Selected for the FastAPI Backend

The FastAPI backend is a persistent application that exposes REST APIs to both the frontend and the YOLO UNO device. Amazon EC2 allows the team to:

- Install the required Python version and dependencies.
- Run Uvicorn as a `systemd` service.
- Configure backend port `8000`.
- Install CloudWatch Agent.
- Inspect logs and service status through SSH.
- Connect directly to Amazon RDS for PostgreSQL.
- Debug telemetry, command, and acknowledgement requests.

EC2 also makes the workshop easier to demonstrate because participants can observe the backend installation, service management, logging, and troubleshooting steps directly.

The main trade-off is that EC2 is not fully managed. The team remains responsible for operating-system updates, service management, storage review, and network protection.

### Why Amazon RDS for PostgreSQL Was Selected

The project data has a structured and relational form:

- One device produces multiple telemetry records.
- One device can receive multiple commands.
- Each command has a state such as `Pending` or `Executed`.
- The backend queries records by device ID and time.

PostgreSQL is suitable for these structured queries and command-state transitions. Amazon RDS reduces operational work compared with hosting PostgreSQL directly on EC2 and provides built-in integration with CloudWatch metrics.

The trade-off is that an RDS instance continues generating charges according to its running time and provisioned resources, even when workshop traffic is low.

### Why Amazon EBS, VPC, and an IAM Role Were Selected

- **Amazon EBS** supplies the EC2 root volume that persists the operating system, checked-out source, Python virtual environment, and local backend logs across ordinary instance restarts. Capacity, snapshots, encryption, and unattached-volume cost remain operator responsibilities.
- **Amazon VPC** provides explicit subnet, route, and Security Group boundaries. It allows the demo EC2 instance to be reachable while RDS remains private in a DB Subnet Group. This control adds configuration work and requires careful evidence.
- **AWS IAM Role** gives EC2 temporary credentials for CloudWatch publishing without storing AWS access keys in source or `.env`. The role must be limited to the required actions and resources; it does not replace network controls or the CloudWatch Agent.

### Why Amazon CloudWatch Was Selected

CloudWatch centralizes:

- EC2 `CPUUtilization`.
- Memory and disk metrics collected by CloudWatch Agent.
- FastAPI backend logs.
- RDS `CPUUtilization`.
- RDS `DatabaseConnections`.
- CloudWatch Alarm states.

This demonstrates that the system supports both functional operation and infrastructure-level monitoring.

### Why HTTP REST Was Selected Instead of MQTT

HTTP REST was selected for the current `room_01` prototype because the project already uses FastAPI endpoints for telemetry, command polling, and acknowledgement. The same JSON contract can be tested directly with `curl`, PowerShell, browser DevTools, Uvicorn logs, and PostgreSQL queries. This makes each step of the end-to-end flow easy to demonstrate and troubleshoot without adding an MQTT broker, topics, subscriptions, device certificates, and separate message-processing components.

The decision is a scope and simplicity trade-off, not a claim that HTTP is better than MQTT for every IoT system:

| Criterion | HTTP REST in the current prototype | MQTT / AWS IoT Core when scaling |
| :--- | :--- | :--- |
| Integration | Reuses the existing FastAPI request/response API | Requires broker topics, policies, certificates, and subscribers |
| Verification | HTTP status, JSON body, API log, and database row are easy to correlate | Requires inspecting publish/subscribe delivery and broker-side evidence |
| Command delivery | Device periodically polls PostgreSQL-backed `Pending` commands and sends an ACK | Broker can push commands through subscribed topics |
| Connectivity and bandwidth | Repeated polling creates more requests and protocol overhead | Persistent lightweight connections are usually more efficient for many devices |
| Delivery guarantees | Application implements retry, command state, and ACK behavior | MQTT provides QoS levels and session features designed for messaging |
| Current suitability | Adequate for one model room and a small command/telemetry workload | Preferable for larger fleets, intermittent links, lower bandwidth, or event-driven delivery |

For a broader deployment, AWS IoT Core with MQTT should be evaluated together with per-device identity, certificate rotation, topic authorization, QoS, offline behavior, cost, and migration of the existing REST command lifecycle. MQTT remains a future option and is not described as implemented in this Workshop.

### Services Not Used in the Current Version

| Service | Reason It Was Not Selected |
| :--- | :--- |
| **AWS Lambda** | The FastAPI backend is currently deployed as a continuously running EC2 service. Moving to Lambda would require a different deployment and database-connection model |
| **Amazon API Gateway** | Browser API requests use CloudFront `/api/*` to ALB, while YOLO UNO calls the ALB directly. API Gateway is not deployed |
| **Amazon DynamoDB** | The project uses relational data structures and SQLAlchemy models designed for PostgreSQL |
| **AWS IoT Core** | YOLO UNO currently communicates with FastAPI through HTTP REST APIs. MQTT and device certificates remain possible future improvements |
| **Amazon SQS** | The implemented command path uses PostgreSQL `Pending` rows and device polling; no SQS queue, producer, or consumer is deployed |

These services are not excluded because they are unsuitable for IoT. They are outside the current scope so that the workshop can focus on the implemented end-to-end flow between hardware, REST APIs, PostgreSQL, the dashboard, and CloudWatch.

### Cost and Simplicity Considerations

The current architecture prioritizes direct implementation and observability rather than a fully serverless design.

- **Simplicity:** EC2 can host the existing FastAPI backend without splitting it into multiple Lambda functions.
- **Managed services:** RDS reduces database-management work compared with installing PostgreSQL on EC2.
- **Cost:** CloudFront/WAF, S3, ALB, two EC2 instances with EBS, RDS Multi-AZ, and CloudWatch can generate charges and must be included in the clean-up plan.
- **Learning value:** The architecture demonstrates Linux, systemd, REST APIs, PostgreSQL, IAM, Security Groups, and CloudWatch.
- **Scalability:** ALB and ASG provide backend horizontal scaling from 2 to 4 instances; larger device fleets still require authentication, HTTPS for the device route, and possibly an event-driven architecture.

## Verified API contract

FastAPI source in `backend/main.py` and `backend/app/api/` confirms these routes:

| Method | Route | Consumer |
| :--- | :--- | :--- |
| `GET` | `/` | Basic service information |
| `GET` | `/api/health` | Health check |
| `POST` | `/api/telemetry` | YOLO UNO telemetry |
| `GET` | `/api/devices/{device_id}/latest` | Dashboard latest view |
| `GET` | `/api/devices/{device_id}/history` | Dashboard history view |
| `POST` | `/api/devices/{device_id}/commands` | Dashboard control |
| `GET` | `/api/devices/{device_id}/commands/latest` | Device polling |
| `POST` | `/api/devices/{device_id}/commands/{command_id}/ack` | Device ACK |

Firmware supports `MODE_AUTO`, `MODE_MANUAL`, `FAN_ON`, `FAN_OFF`, `LIGHT_ON`, `LIGHT_OFF`, `CURTAIN_OPEN`, and `CURTAIN_CLOSE`. The backend currently accepts any command string because `DeviceCommand` has no active enum validator; unsupported commands remain `Pending` because firmware rejects them without ACK. Do not use the singular `/api/device/...` form.

## Data flows

1. **Telemetry:** YOLO UNO sends camelCase fields to the ALB DNS endpoint → target group selects a healthy FastAPI instance → Pydantic maps fields to snake_case → SQLAlchemy writes `telemetry_logs` in RDS → CloudFront `/api/*` latest/history API → dashboard.
2. **Command:** dashboard through CloudFront `/api/*` → ALB → backend writes `commands.state = "Pending"` → YOLO UNO polls the ALB route → the route named `commands/latest` returns the oldest pending command first (FIFO) → hardware executes it.
3. **ACK:** device sends command ID → backend changes that command to `Executed` → later telemetry reports actuator state. The current ACK service looks up by command ID only; device ownership validation is a known hardening task.
4. **Monitoring:** EC2 default metrics and agent data/logs → CloudWatch; RDS publishes service metrics; alarms evaluate configured thresholds.

Polling can make `Pending` visible for only a short time. Evidence should therefore include the create-command response and the later `Executed` database/API state.

The database models define `devices`, `telemetry_logs`, and `commands`. Telemetry fields are `temperature`, `humidity`, `light_intensity`, `fan_status`, `light_status`, `curtain_status`, and `timestamp`.

## Security and IAM

### Network Security Table

| Source | Destination | Port | Rule |
| :--- | :--- | :---: | :--- |
| Internet / CloudFront origin traffic | ALB Security Group | 80 | Public API entry point in the verified deployment |
| ALB Security Group | Backend Security Group | 8000 | FastAPI target traffic only |
| `<ADMIN_IP>/32` | Backend Security Group | 22 | Restricted administration when SSH is required |
| Backend Security Group | RDS Security Group | 5432 | PostgreSQL only |
| EC2/RDS | CloudWatch | HTTPS | Outbound monitoring path |

RDS is not publicly accessible. Secrets stay in ignored local files; EC2 uses an IAM Role instead of hard-coded AWS keys. CloudFront provides viewer HTTPS, ALB/ASG provides backend availability, and RDS is Multi-AZ. The current evidence does not prove ALB HTTPS, application authentication, rate limiting, or WAF blocking mode.

### Security and IAM Table

| Control | Current implementation | Evidence to retain | Limitation / next hardening step |
| :--- | :--- | :--- | :--- |
| Administrator access | SSH 22 restricted to `<ADMIN_IP>/32` | EC2 Security Group rule | Review/rotate key ownership; consider managed access |
| Database isolation | RDS private; 5432 accepts the EC2 Security Group only | RDS setting, subnet group, SG reference | Review NACL/routes and TLS verification |
| AWS credentials | EC2 IAM Role with `CloudWatchAgentServerPolicy` for monitoring | Instance profile and attached policy | Replace broad managed permissions with reviewed resource-scoped policy where practical |
| Application secrets | `.env` and `hardware/include/secrets.h` remain local and ignored | Redacted file locations and `git status` | Move production secrets to an approved managed secret service |
| Public API | CloudFront `/api/*` forwards to ALB HTTP 80; ALB forwards to backend port 8000 | CloudFront behavior, ALB listener, target health, and health request | Add ALB TLS/custom domain plus authentication, authorization, and rate limiting before production |
| Database identity | Dedicated PostgreSQL user in `DATABASE_URL` | Redacted connection configuration | Restrict grants, rotate password, audit access |

### Principle of Least Privilege

Grant only the actions needed by each identity, limit network sources and destinations, and avoid permanent credentials. The workshop does not require `AdministratorAccess`: the human operator needs only the approved provisioning/inspection actions, EC2 needs only monitoring publication permissions, RDS accepts PostgreSQL only from the EC2 Security Group, and the database user should receive only application-required privileges. Any temporary broad permission used during troubleshooting must be documented, time-bounded, reviewed, and removed.

## Current Operational Model

- CloudFront serves the private-S3 React/Vite frontend and routes browser `/api/*` traffic to the ALB.
- The ASG keeps two Amazon Linux FastAPI/Uvicorn instances in service across `ap-southeast-1a` and `ap-southeast-1c` with scaling limits of 2 to 4.
- Each backend instance uses an encrypted gp3 EBS root volume and the `aws-iot-backend` systemd service.
- Private RDS PostgreSQL Multi-AZ stores devices, telemetry, and command states; the standby supports failover, not reads.
- YOLO UNO calls the ALB DNS endpoint directly through periodic HTTP polling and acknowledgement.
- CloudWatch Agent and native AWS metrics feed backend logs, the operations dashboard, ALB/ASG metrics, and eight alarms; the verified alarms currently have no actions.

## Future Scalability Options and Current Limitations

The `device_id` route structure and relational schema can support more rooms, but the present acceptance scope is `room_01`. ALB/ASG removes the single compute endpoint, while periodic device polling still introduces delay and the direct device route still lacks verified TLS and authentication.

Possible future options include ALB HTTPS with a custom domain, authentication and per-device authorization, managed MQTT through AWS IoT Core, queue-based processing such as SQS, caching, read replicas, containers, and Infrastructure as Code. These require architecture, cost/security review, implementation, and tests.

**Amazon SQS and an event-driven architecture are not deployed in the current project.** Auto Scaling is already implemented for the backend.

## Expected result and troubleshooting

Every arrow in the architecture should map to one API call, network rule, database action, or metric/log path. If a connection is unclear, identify its source, destination, port, identity, and expected evidence before provisioning.

Next: [build the AWS foundation](../5.4-AWS-Infrastructure-Setup/).
