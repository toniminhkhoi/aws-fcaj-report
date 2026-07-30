---
title: "Tuần 1 - Phân tích yêu cầu và lập kế hoạch"
date: "2026-06-01"
weight: 1
chapter: false
pre: " <b> 1.1. </b> "
---

> **Thời gian:** 01/06/2026 – 07/06/2026
> **Vai trò:** Tham gia phân tích yêu cầu; phụ trách góc nhìn AWS và phần cứng trong kế hoạch chung của nhóm.

## Mục tiêu

- Xác định bài toán, đối tượng sử dụng và phạm vi của mô hình thử nghiệm.
- Thống nhất sản phẩm bàn giao, tiêu chí thành công và trách nhiệm của từng thành viên.
- Phác thảo kiến trúc ban đầu cho luồng telemetry và command.

## Công việc đã thực hiện

| Hạng mục | Công việc đã thực hiện | Kết quả/Bằng chứng |
| :--- | :--- | :--- |
| Phân tích yêu cầu | Cùng nhóm xác định nhu cầu giám sát nhiệt độ, độ ẩm, ánh sáng và điều khiển quạt, đèn, rèm trong Smart Room | Danh sách chức năng telemetry, latest/history, command và ACK |
| Xác định phạm vi | Thống nhất sử dụng `room_01` làm `device_id` để định danh phòng mẫu | Phạm vi và tiêu chí nghiệm thu được ghi trong Proposal |
| Xác định người dùng | Phân tích nhu cầu của người học AWS, người vận hành phòng, người bảo trì và người đánh giá FCAJ | Danh sách đối tượng sử dụng và giá trị nhận được của từng nhóm |
| Thiết kế kiến trúc | Phác thảo luồng YOLO UNO → FastAPI trên EC2 → RDS PostgreSQL → React Dashboard, kết hợp CloudWatch | Sơ đồ kiến trúc ban đầu và luồng dữ liệu hai chiều |
| Phân công và lập kế hoạch | Xác định trách nhiệm AWS/Hardware, Backend, Frontend/Integration và Documentation/QA; chia dự án thành 8 tuần | Bảng phân công, timeline 01/06–31/07 và danh sách rủi ro ban đầu |
## Kết quả tuần

- Hoàn thành phạm vi, phân công và kiến trúc ban đầu của dự án.
- Xác định các tiêu chí có thể đo như HTTP 200 cho health check, telemetry được lưu, command chuyển `Pending` → `Executed` và log/metric xuất hiện trên CloudWatch.
- Thống nhất chỉ tuyên bố những thành phần có mã nguồn hoặc bằng chứng triển khai.

## Khó khăn và bài học

Khó khăn ban đầu là chuyển ý tưởng Smart Room thành các yêu cầu và tiêu chí kiểm thử cụ thể. Bài học rút ra là mỗi chức năng cần có đầu ra quan sát được để nhóm có thể kiểm tra và thu thập bằng chứng.

## Liên kết Workshop

- [5.1 Tổng quan Workshop](../../5-workshop/5.1-workshop-overview/)
- [5.11 Kết quả, thách thức và hướng cải tiến](../../5-workshop/5.11-results-challenges-future/)
