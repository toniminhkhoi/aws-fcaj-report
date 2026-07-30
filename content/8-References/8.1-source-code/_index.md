---
title: "Source Code"
date: "2026-07-30"
weight: 1
chapter: false
pre: " <b> 8.1. </b> "
---

The application source code is maintained in a separate GitHub repository.

## Main repository

- **Repository:** [AWS IoT Monitoring and Control Dashboard](https://github.com/toniminhkhoi/aws-iot-dashboard)
- **Handover branch:** `main`
- **Access:** Public or otherwise accessible to the project reviewers

## Main components

| Directory | Content |
| :--- | :--- |
| `backend/` | FastAPI backend, REST APIs, and PostgreSQL integration |
| `frontend/` | React + Vite dashboard |
| `hardware/` | PlatformIO firmware for YOLO UNO |
| `diagrams/` | Architecture and hardware diagrams |
| `README.md` | English project instructions |
| `README.vi.md` | Vietnamese project instructions |

## Repository security

The repository must not contain:

- `.env`
- `secrets.h`
- database passwords
- Wi-Fi passwords
- AWS access keys
- SSH private keys
- private credentials or tokens

Template files such as `.env.example` and `secrets.example.h` may be committed, but they must not contain real secret values.