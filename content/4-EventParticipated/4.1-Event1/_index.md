---
title: "Event 1"
date: "2025-09-09"
weight: 1
chapter: false
pre: " <b> 4.1. </b> "
---

# Report on “First Cloud AI Journey Community Day”

### Purpose of the Event

- Share practical career journeys, moving from basic IT roles to Cloud and DevOps engineering.
- Explore the implementation of GraphRAG applications using Amazon Bedrock and Amazon Neptune.
- Understand the golden rules of effective teamwork and collaboration tools.
- Learn how to build multiplayer game architectures in the cloud connecting Godot clients with AWS WebSockets.
- Discover how to combine AWS WAF with Machine Learning for Network Intrusion Detection Systems (NIDS).
- Introduce Docker as a containerization technology and compare it with traditional virtualization.

### Speakers

- **Tran Trung Vinh** – System Administrator at Central Retail Group
- **Viet Phat** – AI Major at Swinburne University of Technology
- **Truong Huy Phuoc** – Presenter on Teamwork
- **Nguyen Quoc Bao** – Presenter on Multiplayer in the Cloud
- **Le Hoang Gia Dai** – Final-year student at HUTECH University / AWS G3
- **Bao Huynh** – Junior Cloud Native Developer at Endava Vietnam / Founder of ITea Lab

### Key Highlights

## Main Content

1. **From IT Helpdesk to Senior Sysadmin & Cloud/DevOps**
   - The transition to Cloud requires adopting a new mindset involving AWS, elastic scaling, and managed services, replacing manual on-premise configurations.
   - Infrastructure as Code (IaC) using Terraform allows for repeatable deployments and version control of infrastructure.
   - A modern DevOps culture involves CI/CD pipelines, Docker, automation, and close collaboration with development teams.

2. **Building GraphRAG Applications**
   - Traditional RAG augments LLMs with external knowledge, but GraphRAG adds relationship awareness and multi-hop reasoning through graph traversal.
   - **Fully Managed Route:** Uses Amazon Bedrock Knowledge Bases for chunking, entity extraction, and embeddings, paired with Amazon Neptune Analytics for graph storage.
   - **Custom Route:** Utilizes LlamaIndex for the processing pipeline and Amazon Neptune to store the Knowledge Graph and execute Cypher queries.

3. **The Art of Effective Teamwork**
   - The efficiency of teamwork is driven by 4 Golden Rules: Clear & Shared Goals, Right Person in the Right Place, Open Communication & Active Listening, and Personal Accountability.
   - Recommended digital tools for team management and communication include Trello, ClickUp, Google Workspace, Slack, and Discord.

4. **Multiplayer in the Cloud (Godot + AWS)**
   - Turn-based or lobby games can be efficiently built using WebSockets, which provide real-time, full-duplex, and reliable communication.
   - The AWS Architecture integrates API Gateway WebSockets, AWS Lambda (Node.js) for game logic, and DynamoDB for storing connection states and player choices.
   - Challenges include stale connections (GoneException) and DynamoDB scan costs; AWS GameLift is recommended for continuous synchronization and authoritative game states in memory.

5. **Machine Learning-based NIDS on AWS**
   - Traditional AWS WAF relies on predefined rules, which struggle against novel, zero-day, or hybrid attack techniques.
   - A Machine Learning model trained on the CSE-CIC-IDS2018 dataset can detect unprecedented anomalous behaviors proactively.
   - The system architecture integrates AWS WAF, Amazon EC2, Application Load Balancer, and ML models like LightGBM and Random Forest to achieve high accuracy.

6. **Docker and Containerization**
   - Containerization packages an application with all its dependencies so it runs consistently across any environment (physical, virtual, or cloud).
   - Unlike Virtual Machines that are heavyweight, require their own OS, and take minutes to boot, Docker containers share the host OS, boot in milliseconds, and use fewer resources.
   - Docker uses layers from Dockerfiles; unchanged layers are reused from cache during builds to speed up the process.

### Key Learnings

- **Design & Operational Mindset**
  - Never test in production—protect availability, trust, and your team's time.
  - Automate repetitive tasks to save time and document configurations and runbooks clearly.
  - A real portfolio and hands-on experience matter far more than just holding certifications.

- **Technical Architecture**
  - When building serverless multiplayer games, be aware that AWS Lambda is stateless, meaning game state must be stored and retrieved from DynamoDB on every request.
  - Signature-based protection alone is insufficient for modern security; ML-based NIDS effectively complements traditional firewalls like AWS WAF.
  - Utilizing containerization (Docker) is ideal for CI/CD pipelines, microservices, and modernizing legacy applications.

- **Data & AI Strategy**
  - In Machine Learning, data quality is critical; handling class imbalance is necessary to improve the detection of minority attack classes.
  - GraphRAG approaches overcome standard LLM limitations by providing explicit relationship storage through nodes and edges.

### Application to Work

- **In infrastructure & operations**:
  - Adopt Infrastructure as Code (Terraform) to define virtual infrastructure and databases instead of doing manual configurations.
  - Implement system monitoring before incidents happen and maintain an operational mindset focused on preventive care.

- **In software development & AI**:
  - Package applications using Docker to eliminate "it works on my machine" compatibility issues.
  - Experiment with Amazon Bedrock and Neptune to extract entities and build Knowledge Graphs for more accurate LLM responses.

- **In teamwork**:
  - Apply the "Right Person, Right Place" rule and leverage tools like ClickUp or Slack to streamline project tracking and communication.

#### Event Photos

<img src="/images/4-EventParticipated/image_1.jpg" alt="Event 1" width="600"/>
