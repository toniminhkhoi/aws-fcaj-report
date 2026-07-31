---
title: "Self-Assessment"
date: "2026-06-15"
weight: 6
chapter: false
pre: " <b> 6. </b> "
---

During the project period from **01/06/2026** to **31/07/2026**, I applied knowledge from school to build and operate the **AWS IoT Monitoring and Control Dashboard** prototype. My main responsibilities focused on AWS infrastructure, system monitoring, YOLO UNO hardware integration, and technical documentation.

### Contribution Summary

- Helped evolve the system from a basic EC2 and RDS deployment into an architecture using CloudFront, private S3, WAF monitoring, ALB, Auto Scaling Group, RDS PostgreSQL Multi-AZ, and CloudWatch.
- Configured and validated networking, security groups, IAM roles, health checks, operational metrics, alarms, logs, backup settings, and deployment evidence.
- Integrated the YOLO UNO/ESP32-S3 device with sensors and actuators for telemetry submission, command polling, and command acknowledgement over HTTP REST.
- Coordinated with backend, frontend, and documentation members to verify the end-to-end data and control flow.
- Updated the workshop with screenshots, validation results, known limitations, and handover notes.

### Self-Assessment

| No. | Criteria | Level | Evidence from the project |
| :-- | :-- | :-- | :-- |
| 1 | **Professional knowledge and skills** | Good | Applied core AWS, networking, monitoring, database, and embedded-system knowledge to a working prototype. |
| 2 | **Learning ability** | Good | Learned unfamiliar AWS services and validated configurations through documentation and practical testing. |
| 3 | **Proactivity** | Good | Researched deployment options, collected evidence, and proposed architecture improvements when limitations were found. |
| 4 | **Sense of responsibility** | Good | Followed assigned infrastructure and hardware tasks through implementation, validation, and documentation. |
| 5 | **Discipline and planning** | Fair | Completed the main scope, but some changes were made close to the documentation deadline and required rework. |
| 6 | **Receptiveness to feedback** | Good | Revised the architecture, workshop content, and validation steps after reviews. |
| 7 | **Communication** | Fair | Shared technical progress with the team, but status updates and explanations were sometimes longer or less structured than necessary. |
| 8 | **Teamwork** | Good | Coordinated integration points and supported verification across infrastructure, backend, frontend, and hardware. |
| 9 | **Professional conduct** | Good | Maintained a cooperative attitude, respected team responsibilities, and documented technical decisions. |
| 10 | **Problem-solving** | Fair | Resolved several deployment and integration issues, but troubleshooting could be more systematic and better recorded. |
| 11 | **Contribution to the project** | Good | Delivered the AWS deployment, monitoring evidence, hardware integration, and a substantial part of the workshop documentation. |
| 12 | **Overall** | Good | Achieved the prototype objectives while identifying clear technical and working-process improvements. |

### Key Lessons Learned

- A cloud architecture should be evaluated not only by whether it runs, but also by security boundaries, availability, observability, backup, cost, and recovery capability.
- Operational claims should be supported by reproducible evidence such as health checks, API responses, database records, metrics, logs, and screenshots.
- Hardware, backend, database, and cloud services must use a consistent data contract for telemetry, commands, and acknowledgement states.
- High availability requires both suitable AWS configuration and controlled failure testing; configuration screenshots alone are not sufficient proof of recovery behavior.

### Areas for Improvement

- Plan infrastructure changes earlier, document the expected impact, and prepare a rollback path before applying them.
- Report progress, blockers, risks, and decisions in a shorter and more structured format.
- Use a repeatable troubleshooting checklist and record the root cause instead of relying on trial and error.
- Learn Infrastructure as Code, automated deployment, and integration testing to reduce manual configuration drift.
- Deepen knowledge of HTTPS origin security, authentication, least-privilege IAM, WAF enforcement, and secret management.
- Run controlled Auto Scaling and RDS failover exercises when the environment and budget allow.
