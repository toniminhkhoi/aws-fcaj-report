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

| Thời gian | Công việc | Kết quả ghi nhận |
| :--- | :--- | :--- |
| 08–09/06 | So sánh EC2/RDS với các lựa chọn serverless và IoT được quản lý | Chọn EC2, EBS, RDS, VPC, IAM Role và CloudWatch; ghi rõ Lambda, API Gateway, DynamoDB, S3 và AWS IoT Core chưa được sử dụng |
| 10/06 | Thiết kế VPC, public subnet cho EC2 và DB Subnet Group dùng các subnet riêng | RDS không cần truy cập công khai; EC2 là thành phần được phép kết nối PostgreSQL |
| 11/06 | Thiết kế Security Group cho SSH, API demo và kết nối EC2 → RDS | SSH giới hạn theo IP quản trị; RDS 5432 chỉ nhận từ EC2 Security Group |
| 12/06 | Xác định IAM Role cho CloudWatch Agent | EC2 dùng quyền tạm thời thay vì hard-code AWS access key |
| 13–14/06 | Đối chiếu sơ đồ với luồng API, cơ sở dữ liệu và metric/log | Mỗi kết nối có nguồn, đích, cổng, danh tính và bằng chứng cần thu thập |

## Kết quả tuần

- Hoàn thành sơ đồ kiến trúc và ranh giới dịch vụ AWS.
- Có kế hoạch mạng, Security Group và IAM phù hợp với mô hình thử nghiệm.
- Phân biệt rõ mô hình vận hành hiện tại với các lựa chọn tương lai như Auto Scaling, SQS hoặc kiến trúc hướng sự kiện.

## Khó khăn và bài học

Một sơ đồ kiến trúc chỉ có giá trị khi từng mũi tên tương ứng với kết nối hoặc quyền thực tế. Việc chọn ít dịch vụ hơn giúp nhóm dễ triển khai, kiểm thử và giải thích các đánh đổi.

## Liên kết Workshop

- [5.3 Kiến trúc và thiết kế dịch vụ](../../5-workshop/5.3-architecture-and-service-design/)
- [5.4 Thiết lập hạ tầng AWS](../../5-workshop/5.4-aws-infrastructure-setup/)
