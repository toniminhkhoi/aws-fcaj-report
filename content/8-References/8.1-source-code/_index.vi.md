---
title: "Source Code"
date: "2026-07-30"
weight: 1
chapter: false
pre: " <b> 8.1. </b> "
---

Source code của ứng dụng được quản lý trong một GitHub repository riêng.

## Repository chính

- **Repository:** [AWS IoT Monitoring and Control Dashboard](https://github.com/toniminhkhoi/aws-iot-dashboard)
- **Nhánh bàn giao:** `main`
- **Quyền truy cập:** Công khai hoặc được cấp quyền phù hợp cho người chấm

## Thành phần chính

| Thư mục | Nội dung |
| :--- | :--- |
| `backend/` | FastAPI backend, REST API và kết nối PostgreSQL |
| `frontend/` | React + Vite dashboard |
| `hardware/` | Firmware PlatformIO cho YOLO UNO |
| `diagrams/` | Sơ đồ kiến trúc và sơ đồ phần cứng |
| `README.md` | Hướng dẫn tiếng Anh |
| `README.vi.md` | Hướng dẫn tiếng Việt |

## Bảo mật repository

Repository không được chứa:

- `.env`
- `secrets.h`
- mật khẩu cơ sở dữ liệu
- mật khẩu Wi-Fi
- AWS access key
- SSH private key
- thông tin xác thực hoặc token riêng tư

Các file mẫu như `.env.example` và `secrets.example.h` có thể được commit nhưng không được chứa giá trị thật.