# Workshop Update Summary

## 1. Preservation approach

The original Workshop structure and instructional content were restored from `HEAD` before this update. The revision then changed only statements, commands, diagrams, tables, and evidence that no longer matched the deployed AWS environment.

- No Workshop chapter was deleted.
- No bilingual page was replaced by a shorter rewrite.
- Existing backend, database, firmware, frontend, testing, monitoring, clean-up, results, and handover procedures were retained when still correct.
- Local frontend execution remains documented as a development path; it is no longer described as the production architecture.
- The original `IoT_Dashboard_Architecture.png` is retained and is used by the Workshop and web Proposal as requested.

## 2. Current architecture used as the source of truth

- Region: `ap-southeast-1`.
- Browser: HTTPS → CloudFront + AWS WAF.
- Static frontend: CloudFront default behavior → private S3 through OAC.
- Browser API: CloudFront `/api/*` → ALB HTTP:80.
- Device API: YOLO UNO → ALB DNS directly over HTTP.
- Backend: ALB → target group HTTP:8000 → two FastAPI/Uvicorn EC2 instances in an ASG.
- ASG capacity: minimum/desired/maximum `2/2/4`, with instances in `ap-southeast-1a` and `ap-southeast-1c`.
- Storage: 10 GiB gp3 EBS root volumes encrypted with the AWS-managed `aws/ebs` KMS key.
- Database: private RDS PostgreSQL Multi-AZ, primary in `ap-southeast-1c`, standby in `ap-southeast-1b`.
- Recovery: seven-day automated backup retention plus an available manual snapshot.
- Security: backend port `8000` accepts the ALB Security Group; RDS port `5432` accepts the backend/EC2 Security Group.
- WAF: three AWS managed rule groups in Count/Monitor mode.
- Monitoring: ALB, ASG, EC2, and RDS dashboard metrics plus eight CloudWatch alarms; no alarm actions/SNS are configured.

## 3. Proposal review

Both Proposal pages retain the original 12-section structure. The following corrections were applied:

- Replaced the old local-frontend/single-EC2/Single-AZ description with the deployed CloudFront/WAF/private-S3, ALB/ASG, and RDS Multi-AZ architecture.
- Separated the browser path from the direct YOLO UNO-to-ALB path.
- Updated objectives, scope, resource placement, data flows, implementation phases, Week 8 milestone, resource/cost table, risks, success criteria, limitations, and deliverables.
- Corrected security claims: viewer HTTPS is enabled, but ALB-origin/device routes remain HTTP; WAF is Count/Monitor, not Block.
- Corrected availability claims: two healthy backend targets and RDS primary/standby are deployed, but a controlled failover drill is not documented.
- Corrected monitoring claims to eight alarms with no notification action.
- Added a visible note that the downloadable Proposal PDF must be re-exported before final submission.

## 4. Workshop chapter updates

| Chapter | Preserved content | Focused correction/addition |
|---|---|---|
| 5.1 | Objectives, audience, outputs, acceptance framing | Current architecture, production path, T01–T14 scope |
| 5.2 | Accounts, tools, permissions, safety | CloudFront/WAF/S3/ELB/ASG permissions and real tool-version evidence |
| 5.3 | Service selection, API contract, data model, security tables | Current architecture SVG, separate browser/device flows, ALB/ASG, Multi-AZ, encrypted EBS |
| 5.4 | VPC, subnet, SG, IAM, EC2, RDS, verification, troubleshooting | Private S3/OAC, CloudFront behaviors, WAF Count mode, AMI/Launch Template/ASG, ALB, target health, backups |
| 5.5 | Backend deployment, systemd, RDS connection, schema checks | ALB health check, RDS endpoint/failover wording, telemetry-to-database evidence |
| 5.6 | Wiring, firmware configuration, telemetry, polling, ACK | Firmware endpoint changed from instance IP to direct ALB DNS |
| 5.7 | Local development, telemetry cards, controls, history, recommendations | Production private-S3/CloudFront path, relative `/api`, disabled API cache, production API evidence |
| 5.8 | Existing test matrix and fan/light/curtain validation | CloudFront production route, ALB target-health test, end-to-end and `Pending` → `Executed` evidence |
| 5.9 | Agent installation, logs, metrics, alarm procedure | ALB/ASG metrics, operations dashboard, eight alarms, explicit “No actions” limitation |
| 5.10 | Cost, security review, clean-up procedure | Complete dependency order for CloudFront/WAF/S3, ALB/TG/ASG/AMI/EBS, RDS/snapshot, CloudWatch |
| 5.11 | Results, challenges, team reflections, future work | Current results, T01–T14, eight alarms, HTTP/WAF/auth/failover limitations |
| 5.12 | Repository, runbook, secrets, handover checklist | ALB/CloudFront checks, immutable AMI/Launch Template/ASG release path, current operating limitations |

## 5. Image decisions

### KEEP

Existing evidence that remains valid was kept, including:

- backend `systemd` and local health check;
- PostgreSQL tables and command records;
- dashboard controls, analysis, and history;
- physical fan, LED, and curtain validation;
- EC2/RDS metrics and backend CloudWatch logs;
- YOLO UNO hardware and firmware-build evidence;
- repository handover structure.

### REPLACE

- The primary Workshop and Proposal architecture references use the original `IoT_Dashboard_Architecture.png`; the newer SVG remains available as a supplementary artifact.
- The CloudWatch alarm screenshot was updated to the current eight-alarm view.

### ADD

- Architecture and separate data-flow SVGs.
- Private S3/OAC, CloudFront origins/behaviors, and WAF managed-rule evidence.
- Security Group chain, IAM Role, AMI/Launch Template, ASG, encrypted EBS, ALB listener, healthy target group, Multi-AZ, backup-retention, and manual-snapshot evidence.
- ALB health, telemetry/database correlation, CloudFront API responses, end-to-end validation, command lifecycle, and operations-dashboard evidence.

### REMOVE

No original screenshot or Workshop section was intentionally removed. Only obsolete claims were removed or rewritten, including:

- production frontend “runs locally”;
- direct public EC2 IP as the application endpoint;
- S3 and Auto Scaling “not implemented”;
- single backend/Single-AZ descriptions;
- unencrypted EBS;
- five-alarm or six-alarm counts;
- WAF described as blocking;
- standby described as a read replica;
- unsupported end-to-end HTTPS, failover, notification, or machine-learning claims.

## 6. Evidence still required or intentionally marked TODO

- Re-export the downloadable English and Vietnamese Proposal PDFs from the revised web Proposal.
- Add direct RDS Console evidence showing `Multi-AZ: Yes` if the evaluator requires a console field in addition to the CLI primary/standby evidence.
- Add one YOLO UNO serial-monitor capture that visibly correlates telemetry POST, command polling, physical execution, and ACK.
- Run and document a controlled ASG/ALB recovery test and RDS Multi-AZ failover drill before claiming tested failover.
- Configure and verify ALB-origin/device TLS, authentication, WAF Block mode, and SNS/notification actions before describing them as implemented.

## 7. Assumptions

- The screenshots supplied for 31 July 2026 represent the intended final AWS environment.
- Redacted account IDs, DNS fragments, instance IDs, Security Group IDs, and administrator IPs are not reconstructed.
- RDS standby is used for managed failover only.
- Rule-based recommendations are deterministic application logic, not a trained AI/ML model.
- Hugo was not available on the current PATH, so source-level checks were completed but a fresh Hugo render was not produced in this pass.

## 8. Validation completed

- Confirmed all 13 English Workshop pages have Vietnamese counterparts.
- Confirmed heading and image counts match for every English/Vietnamese Workshop pair.
- Checked image references under the revised Workshop/Proposal scope.
- Parsed both new SVG files as valid XML.
- Ran `git diff --check`; no whitespace errors were reported.
- Searched the revised scope for stale single-EC2, Single-AZ, local-production, unencrypted-EBS, obsolete alarm-count, and direct-instance-endpoint claims.
