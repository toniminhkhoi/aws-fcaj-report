---
title: "Source Code"
date: "2026-07-30"
weight: 1
chapter: false
pre: " <b> 8.1. </b> "
---

The application source code is maintained in the project's GitHub repository.

## Main repository

- **Repository:** [AWS IoT Monitoring and Control Dashboard](https://github.com/toniminhkhoi/aws-iot-dashboard)
- **Handover branch:** `main`
- **Access at handover:** Public

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

The following files and values must not be committed to the repository:

- `.env`
- `secrets.h`
- database passwords
- Wi-Fi passwords
- AWS access keys
- SSH private keys
- private credentials or tokens

Template files such as `.env.example` and `secrets.example.h` may be committed, but they must contain example values only. Real configuration files must be excluded through `.gitignore` and checked again before handover.
