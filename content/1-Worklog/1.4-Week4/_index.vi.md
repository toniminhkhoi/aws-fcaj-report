---
title: "Tuần 4 - Xây dựng nền tảng backend và cơ sở dữ liệu"
date: "2026-06-22"
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

> **Thời gian:** 22/06/2026 – 28/06/2026
> **Vai trò:** Phụ trách môi trường EC2 và vận hành dịch vụ; phối hợp với thành viên backend để kết nối PostgreSQL.

## Mục tiêu

- Chuẩn bị môi trường Python và triển khai cấu trúc FastAPI trên EC2.
- Kết nối backend với cơ sở dữ liệu `iot_dashboard`.
- Chạy Uvicorn ổn định bằng `systemd`.

## Công việc đã thực hiện

| Hạng mục | Công việc đã thực hiện | Kết quả/Bằng chứng |
| :--- | :--- | :--- |
| Chuẩn bị EC2 | Cài Git, Python, PostgreSQL client và các công cụ cần thiết trên Amazon Linux | EC2 có đủ môi trường để chạy và chẩn đoán backend |
| Triển khai ứng dụng | Sao chép repository, tạo `venv`, cài `requirements.txt` và xác nhận entry point `main:app` | FastAPI chạy được bằng Uvicorn trên EC2 |
| Quản lý cấu hình | Tạo `.env` cục bộ với `DATABASE_URL` và loại trừ thông tin bí mật khỏi Git | Backend kết nối RDS mà không hard-code credential trong mã nguồn |
| Khởi tạo database | Chạy `app.database.init_db` và kiểm tra schema bằng PostgreSQL client | Các bảng `devices`, `telemetry_logs` và `commands` được tạo bằng SQLAlchemy `create_all` |
| Vận hành backend | Tạo service `aws-iot-backend`, cấu hình log, khởi động lại dịch vụ và kiểm tra `/api/health` | Service ở trạng thái `active (running)` và health check trả HTTP 200 |
## Kết quả tuần

- FastAPI chạy ổn định trên EC2 dưới sự quản lý của `systemd`.
- Backend kết nối được với RDS PostgreSQL riêng.
- Xác nhận dự án chưa có quy trình migration bằng Alembic; không ghi nhận Alembic như thành phần đã triển khai.

## Khó khăn và bài học

Đường dẫn, tài khoản Linux, file môi trường và module Uvicorn phải khớp chính xác giữa terminal và service file. `journalctl`, file log và health check là ba nguồn quan trọng để chẩn đoán lỗi khởi động.

## Liên kết Workshop

- [5.5 Triển khai backend và tích hợp cơ sở dữ liệu](../../5-workshop/5.5-backend-and-database/)
- [5.12 Bàn giao dự án](../../5-workshop/5.12-project-handover/)
