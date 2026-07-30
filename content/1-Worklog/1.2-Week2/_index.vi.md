---
title: "Tuần 2 - Thiết kế kiến trúc AWS và nền tảng mạng"
date: "2026-06-08"
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---

> **Thời gian:** 08/06/2026 – 14/06/2026
> **Vai trò:** Phụ trách thiết kế AWS, mạng và IAM; phối hợp với nhóm backend để xác định luồng API và dữ liệu.

## Mục tiêu

- Chọn các dịch vụ AWS phù hợp với mã nguồn và phạm vi Workshop.
- Thiết kế VPC, subnet, Security Group và IAM Role theo nguyên tắc đặc quyền tối thiểu.
- Xác định luồng telemetry, command, ACK và giám sát.

## Công việc đã thực hiện

| Hạng mục | Công việc đã thực hiện | Kết quả/Bằng chứng |
| :--- | :--- | :--- |
| Lựa chọn hạ tầng | Xác định các thành phần cần thiết cho Smart Room và thống nhất sử dụng EC2, EBS, RDS, VPC, IAM Role và CloudWatch | Danh sách dịch vụ và vai trò của từng thành phần trong kiến trúc |
| Thiết kế mạng | Thiết kế VPC, public subnet cho EC2 và DB Subnet Group từ các subnet dành cho database | Sơ đồ mạng đặt EC2 và RDS trong đúng ranh giới kết nối |
| Security Group | Xác định quy tắc cho SSH quản trị, API demo và luồng EC2 → RDS qua PostgreSQL 5432 | Bảng rule giới hạn SSH theo IP quản trị và RDS theo Security Group của EC2 |
| IAM và giám sát | Xác định IAM Role cho EC2 và quyền cần thiết để CloudWatch Agent gửi log/metric | Mô hình dùng quyền tạm thời, không hard-code AWS access key |
| Rà luồng hệ thống | Đối chiếu API, database, cổng mạng, danh tính và dữ liệu giám sát trên sơ đồ | Checklist nguồn, đích, cổng và bằng chứng cần thu thập cho từng kết nối |
## Kết quả tuần

- Hoàn thành sơ đồ kiến trúc và ranh giới dịch vụ AWS.
- Có kế hoạch mạng, Security Group và IAM phù hợp với mô hình thử nghiệm.
- Phân biệt rõ mô hình vận hành hiện tại với các lựa chọn tương lai như Auto Scaling, SQS hoặc kiến trúc hướng sự kiện.

## Khó khăn và bài học

Một sơ đồ kiến trúc chỉ có giá trị khi từng mũi tên tương ứng với kết nối hoặc quyền thực tế. Việc chọn ít dịch vụ hơn giúp nhóm dễ triển khai, kiểm thử và giải thích các đánh đổi.

## Liên kết Workshop

- [5.3 Kiến trúc và thiết kế dịch vụ](../../5-workshop/5.3-architecture-and-service-design/)
- [5.4 Thiết lập hạ tầng AWS](../../5-workshop/5.4-aws-infrastructure-setup/)
