---
title: "Tuần 3 - Triển khai Amazon EC2 và Amazon RDS"
date: "2026-06-15"
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---

> **Thời gian:** 15/06/2026 – 21/06/2026
> **Vai trò:** Phụ trách cấp phát và xác minh hạ tầng AWS.

## Mục tiêu

- Tạo nền tảng mạng và các tài nguyên AWS cần thiết cho backend.
- Khởi chạy EC2, gắn EBS và IAM Role.
- Tạo RDS for PostgreSQL trong mạng riêng và xác minh kết nối từ EC2.

## Công việc đã thực hiện

| Hạng mục | Công việc đã thực hiện | Kết quả/Bằng chứng |
| :--- | :--- | :--- |
| Chuẩn bị môi trường AWS | Chọn `ap-southeast-1`, rà VPC, route, subnet và DB Subnet Group trước khi cấp phát tài nguyên | EC2 và RDS được đặt trong cùng Region và đúng thiết kế mạng |
| Triển khai EC2 và EBS | Khởi chạy `iot-backend-server` loại `t3.micro`, gắn EBS `gp3` 10 GiB và IAM Role `iot-dashboard-cloudwatch-role` | EC2 ở trạng thái `Running`, vượt status check và có role được gắn |
| Triển khai RDS | Tạo `iot-dashboard-db` bằng RDS for PostgreSQL loại `db.t4g.micro` | Database ở trạng thái `Available`, sử dụng DB Subnet Group và không mở truy cập công khai |
| Cấu hình kết nối | Tạo `iot-backend-sg`, `ec2-rds-1`, `rds-ec2-1` và giới hạn PostgreSQL 5432 theo Security Group | EC2 kết nối được RDS mà không mở database trực tiếp ra Internet |
| Xác minh và thu thập bằng chứng | Kiểm tra DNS/cổng 5432 từ EC2, status check, thông tin EBS, RDS và các rule mạng | Ảnh EC2 Running, RDS Available, IAM Role và Security Group phục vụ Workshop |
## Kết quả tuần

- EC2, EBS, RDS, IAM Role và các Security Group đã sẵn sàng cho bước triển khai ứng dụng.
- RDS được giữ trong mạng riêng và không mở PostgreSQL cho `0.0.0.0/0`.
- Thu thập ảnh bằng chứng về EC2, RDS, IAM Role và quy tắc Security Group. Ảnh không chứa mật khẩu hoặc access key; một số định danh tài nguyên và endpoint vẫn cần được cân nhắc che trước khi công khai.

## Khó khăn và bài học

Lỗi kết nối cần được tách thành lỗi mạng và lỗi xác thực. Kiểm tra TCP thành công chỉ chứng minh route và Security Group hoạt động, chưa chứng minh thông tin đăng nhập PostgreSQL đúng.

## Liên kết Workshop

- [5.4 Thiết lập hạ tầng AWS](../../5-workshop/5.4-aws-infrastructure-setup/)
- [5.10 Chi phí, bảo mật và dọn dẹp](../../5-workshop/5.10-cost-security-cleanup/)
