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

| Thời gian | Công việc | Kết quả ghi nhận |
| :--- | :--- | :--- |
| 15/06 | Chọn khu vực `ap-southeast-1`, rà soát VPC, route và DB Subnet Group | EC2 và RDS được đặt trong cùng khu vực và đúng ranh giới mạng |
| 16/06 | Tạo `iot-backend-sg`, `ec2-rds-1` và `rds-ec2-1` | API demo dùng cổng 8000; PostgreSQL 5432 chỉ cho phép luồng từ EC2 tới RDS |
| 17–18/06 | Khởi chạy `iot-backend-server` loại `t3.micro`, gắn EBS `gp3` 10 GiB và IAM Role `iot-dashboard-cloudwatch-role` | EC2 ở trạng thái `Running` và vượt qua các status check |
| 19–20/06 | Tạo `iot-dashboard-db` bằng RDS for PostgreSQL loại `db.t4g.micro` | Cơ sở dữ liệu ở trạng thái `Available`, dùng DB Subnet Group và không mở Internet access gateway |
| 21/06 | Kiểm tra DNS và cổng 5432 từ EC2 tới RDS | Xác nhận route và Security Group cho phép kết nối mạng cần thiết |

## Kết quả tuần

- EC2, EBS, RDS, IAM Role và các Security Group đã sẵn sàng cho bước triển khai ứng dụng.
- RDS được giữ trong mạng riêng và không mở PostgreSQL cho `0.0.0.0/0`.
- Thu thập ảnh bằng chứng về EC2, RDS, IAM Role và quy tắc Security Group sau khi che thông tin nhạy cảm.

## Khó khăn và bài học

Lỗi kết nối cần được tách thành lỗi mạng và lỗi xác thực. Kiểm tra TCP thành công chỉ chứng minh route và Security Group hoạt động, chưa chứng minh thông tin đăng nhập PostgreSQL đúng.

## Liên kết Workshop

- [5.4 Thiết lập hạ tầng AWS](../../5-workshop/5.4-aws-infrastructure-setup/)
- [5.10 Chi phí, bảo mật và dọn dẹp](../../5-workshop/5.10-cost-security-cleanup/)
