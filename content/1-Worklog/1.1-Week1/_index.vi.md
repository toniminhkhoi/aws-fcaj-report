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

| Thời gian | Công việc | Kết quả ghi nhận |
| :--- | :--- | :--- |
| 01–02/06 | Phân tích nhu cầu giám sát nhiệt độ, độ ẩm, ánh sáng và điều khiển quạt, đèn, rèm cho một phòng mẫu | Chốt phạm vi nghiệm thu là thiết bị `room_01`, không mô tả dự án như một BMS đa chi nhánh |
| 03/06 | Xác định người học AWS, người vận hành phòng, người bảo trì và người đánh giá FCAJ là các nhóm sử dụng chính | Mỗi nhóm có nhu cầu và giá trị nhận được được mô tả rõ |
| 04–05/06 | Xây dựng yêu cầu chức năng cho telemetry, dữ liệu mới nhất, lịch sử, command và ACK | Có danh sách đầu ra quan sát được cho từng chức năng |
| 06/06 | Thống nhất vai trò AWS/Hardware, Backend, Frontend/Integration và Documentation/QA | Có bảng phân công để truy vết đóng góp của từng thành viên |
| 07/06 | Phác thảo kiến trúc gồm YOLO UNO, FastAPI trên EC2, RDS PostgreSQL, dashboard React và CloudWatch | Hình thành kiến trúc ban đầu và danh sách rủi ro cần xử lý |

## Kết quả tuần

- Hoàn thành phạm vi, phân công và kiến trúc ban đầu của dự án.
- Xác định các tiêu chí có thể đo như HTTP 200 cho health check, telemetry được lưu, command chuyển `Pending` → `Executed` và log/metric xuất hiện trên CloudWatch.
- Thống nhất chỉ tuyên bố những thành phần có mã nguồn hoặc bằng chứng triển khai.

## Khó khăn và bài học

Khó khăn ban đầu là phạm vi dễ bị mô tả quá lớn so với một mô hình một phòng. Bài học rút ra là phải tách rõ mục tiêu học tập, chức năng đã triển khai và các hướng mở rộng trong tương lai.

## Liên kết Workshop

- [5.1 Tổng quan Workshop](../../5-workshop/5.1-workshop-overview/)
- [5.11 Kết quả, thách thức và hướng cải tiến](../../5-workshop/5.11-results-challenges-future/)
