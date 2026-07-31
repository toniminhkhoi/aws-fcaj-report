---
title: "Sharing and Feedback"
date: "2026-06-15"
weight: 7
chapter: false
pre: " <b> 7. </b> "
---

### Overall Evaluation

The project provided a practical opportunity to connect programming, databases, networking, embedded systems, and AWS services in one end-to-end product. Instead of evaluating each component separately, the team had to make the dashboard, API, database, cloud infrastructure, and physical device work together under a shared data and command flow.

### Learning Environment

The learning environment encouraged self-research and practical validation. Discussions with mentors and other participants helped me identify incomplete assumptions, especially in architecture, security, monitoring, and documentation. The most useful form of support was guidance toward official documentation and verification methods rather than receiving a complete answer immediately.

### Mentor and Team Support

Regular reviews helped the team narrow the project scope, correct technical descriptions, and improve the evidence included in the workshop. Team members divided responsibilities across infrastructure, backend, frontend, hardware, and documentation, then coordinated at integration points. This approach made individual ownership clearer, although integration work still required more frequent status updates.

### Relevance to My Academic Background

The project extended concepts learned at school into a deployed environment. Programming and database knowledge were still essential, but deployment introduced additional concerns such as security groups, IAM, health checks, backup, availability, monitoring, cloud cost, and handover documentation. This experience helped me understand the difference between a local demonstration and a system that can be operated and reviewed by others.

### Skills Developed

- Designing and explaining an AWS architecture with clear traffic and security boundaries.
- Validating an end-to-end IoT flow from telemetry ingestion to database storage and dashboard display.
- Integrating hardware command polling and acknowledgement with backend command states.
- Reading metrics and logs to investigate system behavior instead of relying only on application output.
- Working with teammates through defined responsibilities, integration checkpoints, and shared evidence.
- Writing a technical workshop that distinguishes implemented features, observed results, and future improvements.

### Suggestions for Improvement

- Establish a fixed weekly review schedule so all teams have predictable checkpoints.
- Share the evaluation rubric and required evidence checklist at the beginning of the project.
- Add short architecture, security, and AWS cost reviews before major infrastructure changes.
- Reserve a dedicated integration rehearsal before the final documentation deadline.
- Use a simple shared template for status updates, technical decisions, risks, and change history.

### Personal Takeaway

The most valuable lesson was learning to connect multiple technical layers and prove that they work together with concrete evidence. The project also showed me that a successful prototype still needs honest documentation of its limitations, such as HTTP origin traffic, WAF monitoring mode, limited authentication, and untested failover scenarios. These lessons provide a practical foundation for future study in cloud infrastructure, DevOps, and IoT systems.

> This section is a personal reflection based on the project process. It does not quote mentors or represent an official evaluation by FCAJ.
