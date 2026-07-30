---
title: "Results, Challenges and Future Improvements"
date: "2026-07-28"
weight: 11
chapter: false
pre: " <b> 5.11. </b> "
---

## Overview

This section summarizes the verified project outcomes, project-specific customizations, individual contributions, lessons learned, current limitations, and practical next steps. The conclusions are based on the source code, the T01–T12 test matrix, AWS screenshots, PostgreSQL records, dashboard captures, and the hardware demonstration.

## Achieved Results

| Outcome | Status | Evidence |
| :--- | :---: | :--- |
| FastAPI backend health and service operation | **Pass** | Health response and `systemd` evidence in [section 5.5](../5.5-Backend-and-Database/) |
| Telemetry submission and PostgreSQL persistence | **Pass** | API response and matching RDS record in [section 5.8](../5.8-End-to-End-Testing/) |
| Latest telemetry and ordered history on the dashboard | **Pass** | Dashboard cards, history charts, and HTTP 200 requests in sections [5.7](../5.7-Frontend-Integration/) and [5.8](../5.8-End-to-End-Testing/) |
| Command creation and `Pending` → `Executed` lifecycle | **Pass** | Command ID, ACK response, and PostgreSQL state in sections [5.5](../5.5-Backend-and-Database/) and [5.8](../5.8-End-to-End-Testing/) |
| Physical fan, light, and curtain control | **Pass** | Dashboard-to-hardware captures and the end-to-end demonstration video in [section 5.8](../5.8-End-to-End-Testing/) |
| CloudWatch logs, metrics, and alarms | **Pass** | Backend logs, four dashboard widgets, and five alarm configurations in [section 5.9](../5.9-CloudWatch-Monitoring/) |
| Complete end-to-end validation | **Pass** | All twelve test cases T01–T12 are recorded as Pass in the test matrix |

The test scope covers the implemented Smart Room model identified by `device_id=room_01`. Results are reported only for the demonstrated environment and are not generalized beyond that scope.

## Project Customizations

The project is not an unchanged tutorial deployment. Its reviewed customizations include:

- a `room_01` domain model joining physical telemetry, dashboard history, and actuator state;
- a FastAPI/PostgreSQL command lifecycle with stored `Pending` and `Executed` states plus device ACK;
- 8 firmware commands covering automatic/manual mode and direct fan, light, and curtain control;
- firmware thresholds, GPIO mapping, LCD1602 output, reconnect timing, and ESP32 Preferences-based ACK recovery;
- a React/Vite dashboard with telemetry charts, device controls, and rule-based operating recommendations;
- a private RDS network path through Security Group reference rather than public database access;
- project-specific CloudWatch metrics, backend log collection, and five alarm configurations; and
- an evidence-based bilingual Workshop that connects implementation steps with screenshots, test records, and troubleshooting guidance.

These customizations adapt the architecture to the implemented source code, the YOLO UNO hardware, and the Smart Room use case.

## Individual Contributions

| Contributor | Owned scope and concrete contribution | Evidence path |
| :--- | :--- | :--- |
| **Phạm Lê Minh Khôi** | AWS architecture, network/security boundaries, and EC2/RDS/CloudWatch operations; PlatformIO firmware development for YOLO UNO; sensor, LCD, and actuator integration; telemetry transmission, command polling, execution of all 8 commands, and ACK processing | [Architecture](../5.3-Architecture-and-Service-Design/), [AWS setup](../5.4-AWS-Infrastructure-Setup/), [hardware](../5.6-Hardware-Integration/), [CloudWatch](../5.9-CloudWatch-Monitoring/) |
| **Ngô Minh Thuận** | FastAPI routes, Pydantic aliases, SQLAlchemy models, PostgreSQL persistence, telemetry service, command lifecycle, and ACK processing | [API/data design](../5.3-Architecture-and-Service-Design/), [backend/database](../5.5-Backend-and-Database/), [test matrix](../5.8-End-to-End-Testing/) |
| **Thượng Đình Hưng** | React/Vite dashboard, telemetry visualization, control requests, mode/recommendation UI, integration debugging, and demo-video production | [frontend integration](../5.7-Frontend-Integration/), [end-to-end validation](../5.8-End-to-End-Testing/), [handover](../5.12-Project-Handover/) |
| **Lê Bảo Khánh** | Proposal/report content, blogs/worklogs/events, bilingual Workshop structure, source-to-document verification, navigation, screenshot planning, and QA | [Workshop overview](../5.1-Workshop-overview/), [test evidence](../5.8-End-to-End-Testing/), [results](../5.11-Results-Challenges-Future/), [handover](../5.12-Project-Handover/) |

The linked Workshop sections provide evidence for each area of responsibility. This table records ownership and does not replace the individual reflections below.

## Individual Reflections

### Phạm Lê Minh Khôi

| Reflection field | Reflection |
| :--- | :--- |
| Challenge | Integrate a publicly reachable demo backend, private PostgreSQL, monitoring, and physical actuators without confusing cloud success with hardware success |
| Root Cause | The flow crosses VPC rules, IAM, Linux services, HTTP polling, electrical wiring, and asynchronous ACK state |
| Solution | Use an EC2-to-RDS Security Group reference, EC2 IAM Role, systemd/CloudWatch checks, source-defined GPIOs, safe power, command IDs, and persistent ACK recovery |
| Lesson Learned | Validate each boundary independently and correlate one command ID through API, database, serial output, actuator action, and monitoring |
| Future Improvement | Add HTTPS and a stable endpoint, define infrastructure consistently, tighten IAM permissions, and improve sensor calibration and hardware test records |

### Ngô Minh Thuận

| Reflection field | Reflection |
| :--- | :--- |
| Challenge | Preserve telemetry and make command completion observable across polling and ACK |
| Root Cause | Asynchronous clients and database state can diverge; the current source also lacks command enum validation and strict ACK device ownership |
| Solution | Model devices, telemetry, and commands in PostgreSQL; return command IDs/states; use FIFO pending polling and explicit ACK transitions |
| Lesson Learned | An OpenAPI contract and stored state improve traceability, but validation, authorization, idempotency, and schema migration must be designed explicitly |
| Future Improvement | Add supported-command validation, authenticated device identity, device-bound ACK checks, idempotency rules, Alembic migrations, and automated API tests |

### Thượng Đình Hưng

| Reflection field | Reflection |
| :--- | :--- |
| Challenge | Present live telemetry and controls while accurately distinguishing request acceptance, physical execution, and simulated fallback |
| Root Cause | The current frontend polls multiple endpoints, keeps mode locally, falls back to generated data, and can report mock success after a failed command |
| Solution | Inspect DevTools Network, use plural relative API routes, expose command ID/state, label simulated data, and verify physical execution through ACK/evidence |
| Lesson Learned | A responsive UI is not enough; operational truth must come from backend/device state and error handling must never imply unverified success |
| Future Improvement | Remove false-success fallback, add API-backed mode/command status, centralize environment configuration, correct the Lux label, and add component/integration tests |

### Lê Bảo Khánh

| Reflection field | Reflection |
| :--- | :--- |
| Challenge | Turn evolving source notes and implementation changes into a coherent bilingual Workshop |
| Root Cause | Backend, frontend, firmware, AWS configuration, and evidence were updated at different times by different contributors |
| Solution | Use the active source as the technical reference, align the English and Vietnamese structures, add evidence at the relevant steps, and verify names, paths, tables, and links |
| Lesson Learned | Technical documentation must distinguish implementation, expected behavior, observed results, limitations, and future work while keeping both languages synchronized |
| Future Improvement | Add automated Hugo, link, secret, and bilingual-parity checks; maintain a versioned API/GPIO contract; and schedule a final review by all contributors |

## Key Challenges and Lessons Learned

| Problem | Root cause | Solution | Lesson learned |
| :--- | :--- | :--- | :--- |
| Secure EC2-to-RDS connectivity | RDS must remain private while the backend still needs PostgreSQL access | Allow port 5432 from the EC2 Security Group instead of a public CIDR | Define access by workload identity and required port |
| Command state across API, database, and device | Polling and ACK occur asynchronously | Persist command IDs and states, then correlate the same ID from creation through ACK | A successful API response alone does not prove physical execution |
| Duplicate execution during retries | A device can receive or acknowledge the same command more than once | Track the last command and make ACK retries independent of actuator execution | Retry logic must be designed for idempotency |
| Frontend and backend route mismatch | Singular/plural paths and API targets changed during integration | Use the implemented OpenAPI contract and verify requests in DevTools Network | Maintain one versioned API contract |
| Uncalibrated light readings | The sensor provides a raw analog value | Keep the raw value traceable and document calibration as a follow-up | Do not assign a physical unit without a conversion method |
| Missing CloudWatch datapoints | Agent permissions, dimensions, paths, or collection timing may not match | Check the Agent log, metric dimensions, and actual source path | Alarm state must be interpreted together with the underlying metric data |

## Current Limitations

- The demonstration backend currently uses HTTP on port 8000 and an EC2 public address that may change after a stop/start cycle.
- API routes do not yet enforce strong client or device authentication.
- Command validation and ACK ownership checks should be tightened.
- The frontend still needs stricter handling of simulated fallback and failed command requests.
- The light value is based on an uncalibrated analog reading.
- Two CloudWatch alarms showed `Insufficient data`, and no notification action was attached at the time of capture.

## Future Improvements

- Use a stable domain or endpoint for the deployed backend.
- Add a reviewed reverse proxy, HTTPS, authentication, and stronger authorization.
- Store application secrets in a managed secret solution.
- Support more devices and rooms with ownership/authorization rules.
- Add supported-command validation, device-bound ACK checks, and idempotency rules.
- Introduce versioned database migrations and automated API tests.
- Centralize frontend environment configuration and remove false-success fallback behavior.
- Define infrastructure and deployment steps consistently and automate tested rollback.
- Add a reviewed notification channel for operational alarms.
- Calibrate the light sensor and publish the conversion method/unit.

Each improvement should have an owner, an implementation plan, and test evidence before it is described as part of the current system.

## Result

The project achieved its intended Smart Room scope: telemetry is collected and stored, the dashboard presents current and historical data, commands control the three demonstrated actuators, ACK updates command state, and CloudWatch provides operational evidence. All T01–T12 test cases are recorded as Pass, while the limitations above define the next improvement priorities.

Next: [prepare the project handover](../5.12-Project-Handover/).
