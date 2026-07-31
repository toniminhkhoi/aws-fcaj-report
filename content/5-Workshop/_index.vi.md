---
title: "Workshop"
date: "2026-07-28"
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

Workshop này hướng dẫn xây dựng Smart Room có `device_id=room_01`. CloudFront/WAF phân phối dashboard React từ S3 private và forward request browser `/api/*` tới ALB. ALB phân phối request tới hai FastAPI instance trong ASG; RDS PostgreSQL Multi-AZ lưu telemetry và trạng thái lệnh. YOLO UNO dùng trực tiếp ALB, thực thi lệnh rồi gửi ACK.

## Mục tiêu và kết quả đạt được

Sau khi hoàn thành Workshop, người học có thể:

- chuẩn bị CloudFront/WAF/S3 private, ALB/ASG/EC2/EBS, VPC/Security Group và RDS PostgreSQL Multi-AZ;
- triển khai FastAPI trên EC2 dưới dạng dịch vụ `systemd` và xác minh kết nối cơ sở dữ liệu;
- biên dịch và nạp firmware PlatformIO cho YOLO UNO;
- thu thập telemetry và điều khiển quạt, đèn, rèm thông qua 8 lệnh firmware được hỗ trợ;
- sử dụng dashboard React + Vite để xem dữ liệu hiện tại, theo dõi lịch sử và gửi lệnh điều khiển;
- xác minh vòng đời lệnh từ `Pending` sang `Executed` thông qua ACK của thiết bị;
- kiểm tra log backend, metric ALB/ASG/EC2/RDS và trạng thái alarm trên Amazon CloudWatch; và
- hoàn thành kiểm thử end-to-end, xử lý sự cố, dọn dẹp tài nguyên và bàn giao dự án.

Kết quả cuối cùng là một mô hình Smart Room có thể tái triển khai cho `device_id=room_01`, đi kèm mã nguồn, hướng dẫn triển khai, kết quả kiểm thử, ảnh chụp AWS và video demo phần cứng.

## Nội dung Workshop

1. [5.1 Tổng quan Workshop](5.1-Workshop-overview/)
2. [5.2 Điều kiện tiên quyết](5.2-Prerequisites/)
3. [5.3 Kiến trúc và thiết kế dịch vụ](5.3-Architecture-and-Service-Design/)
4. [5.4 Thiết lập hạ tầng AWS](5.4-AWS-Infrastructure-Setup/)
5. [5.5 Triển khai backend và tích hợp cơ sở dữ liệu](5.5-Backend-and-Database/)
6. [5.6 Tích hợp phần cứng](5.6-Hardware-Integration/)
7. [5.7 Tích hợp frontend](5.7-Frontend-Integration/)
8. [5.8 Kiểm thử và xác minh đầu cuối](5.8-End-to-End-Testing/)
9. [5.9 Giám sát bằng CloudWatch](5.9-CloudWatch-Monitoring/)
10. [5.10 Chi phí, bảo mật và dọn dẹp](5.10-Cost-Security-Cleanup/)
11. [5.11 Kết quả, thách thức và hướng cải tiến](5.11-Results-Challenges-Future/)
12. [5.12 Bàn giao dự án](5.12-Project-Handover/)

## Kiến trúc và luồng hoạt động

![Kiến trúc AWS IoT Monitoring and Control Dashboard](/images/2-Proposal/IoT_Dashboard_Architecture.png)

*Hình 5-1. Kiến trúc hiện tại gồm CloudFront/WAF/S3 private, backend FastAPI qua ALB/ASG, RDS PostgreSQL Multi-AZ, YOLO UNO và CloudWatch.*

Hệ thống hoạt động theo bốn luồng chính:

1. YOLO UNO đọc cảm biến và gửi telemetry trực tiếp qua ALB bằng HTTP.
2. FastAPI kiểm tra dữ liệu rồi lưu telemetry vào Amazon RDS for PostgreSQL.
3. Dashboard trên CloudFront dùng `/api/*` qua ALB để lấy dữ liệu và tạo lệnh.
4. YOLO UNO thăm dò lệnh, điều khiển thiết bị chấp hành tương ứng và gửi ACK; CloudWatch thu thập log cùng metric vận hành.

Môi trường AWS gồm **Amazon S3 private, CloudFront, AWS WAF, ALB, target group, ASG, hai EC2 với EBS mã hóa, RDS PostgreSQL Multi-AZ, VPC/Security Group, IAM, CloudWatch và tám alarm**.

Bắt đầu từ [Tổng quan Workshop](5.1-Workshop-overview/).
