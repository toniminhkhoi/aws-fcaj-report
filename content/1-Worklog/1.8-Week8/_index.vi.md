---
title: "Tuần 8 - Tích hợp, kiểm thử, CloudWatch và bàn giao"
date: "2026-07-20"
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---

> **Thời gian:** 20/07/2026 – 31/07/2026
>
> **Vai trò:** Phối hợp tích hợp toàn hệ thống; phụ trách truy vết lỗi AWS/phần cứng, CloudWatch và kiểm tra bằng chứng kỹ thuật.

## Mục tiêu

- Kiểm tra toàn bộ luồng telemetry và command với phần cứng thực tế.
- Đối chiếu cùng dữ liệu hoặc command ID qua dashboard, API, RDS và firmware.
- Thiết lập log, metric và alarm cho EC2/RDS.
- Hoàn thiện rà soát bảo mật, tài liệu song ngữ, video demo và nội dung bàn giao.

## Công việc đã thực hiện

| Thời gian | Công việc | Kết quả ghi nhận |
| :--- | :--- | :--- |
| 20/07 | Ghi phiên bản mã nguồn, firmware, khu vực AWS và điều kiện kiểm thử | Có mốc cấu hình để tái tạo kết quả |
| 21/07 | Kiểm tra health check, POST telemetry, dữ liệu mới nhất và lịch sử | API và PostgreSQL trả dữ liệu `room_01` nhất quán |
| 22/07 | Tạo command, ghi nhận `Pending`, gửi ACK và truy vấn lại cùng command ID | Cùng bản ghi chuyển sang `Executed` |
| 23–24/07 | Kiểm tra `FAN_*`, `LIGHT_*`, `CURTAIN_*` từ dashboard | Video demo ghi lại phản ứng vật lý của quạt, đèn và servo |
| 25/07 | Gửi payload kiểm soát bằng `curl` và đối chiếu RDS | Xác minh riêng lớp FastAPI → RDS, không dùng ảnh này thay cho bằng chứng phần cứng |
| 26–27/07 | Cấu hình CloudWatch Agent và log group `/aws/ec2/aws-iot-dashboard/backend` | Backend log được thu thập tập trung |
| 28/07 | Tạo dashboard `ec2-rds-metrics` cho EC2 CPU/disk và RDS CPU/connections | Có màn hình theo dõi tài nguyên thực tế |
| 29/07 | Kiểm tra năm alarm EC2/RDS và xử lý trạng thái `Insufficient data` | Ghi rõ memory/disk phụ thuộc agent, namespace, dimension và IAM |
| 30/07 | Rà IAM Role, Security Group, RDS public access, secrets và chi phí | Xác định kiểm soát hiện tại cùng các hạn chế HTTPS, xác thực và High Availability |
| 31/07 | Rà Workshop Anh–Việt, README, caption, test matrix, video và checklist bàn giao | Tài liệu cuối khớp với prototype `room_01` và bằng chứng đã thu thập |

## Kết quả trong tuần

- Các ca kiểm thử telemetry, history, command, polling, actuator và ACK trong ma trận đều đạt.
- Phân biệt rõ bằng chứng API/database với bằng chứng phần cứng.
- CloudWatch nhận backend log, hiển thị metric EC2/RDS và đánh giá năm alarm.
- Hoàn thiện báo cáo song ngữ, Workshop, README, hướng dẫn vận hành và nội dung bàn giao.
- Video demo chứng minh thao tác dashboard và phản ứng vật lý của thiết bị: [Xem video demo](https://drive.google.com/file/d/1T97dUY58hbT2ppxvg7ESR12Jg9BA828W/view?usp=sharing).

## Khó khăn, cách xử lý và bài học

Trạng thái `Pending` tồn tại ngắn khi thiết bị polling nhanh, nên nhóm theo dõi cùng command ID trước và sau ACK. Với CloudWatch, `Insufficient data` được kiểm tra ở agent, namespace, dimension và IAM thay vì xem là trạng thái bình thường. Tôi rút ra rằng một kết quả Pass phải gắn với tiêu chí, kết quả thực tế và bằng chứng; tài liệu cũng phải phân biệt rõ phần đã triển khai với hướng phát triển.

## Giới hạn và hướng phát triển

Mô hình hiện tại dùng HTTP chưa có TLS, API chưa có xác thực và chỉ chạy trên một EC2 cùng một RDS. HTTPS, xác thực người dùng/thiết bị, AWS IoT Core, hàng đợi, High Availability và mở rộng nhiều thiết bị là các phương án tương lai, chưa được triển khai.

## Liên kết Workshop

- [Kiểm thử end-to-end](../../5-workshop/5.8-end-to-end-testing/)
- [Giám sát bằng CloudWatch](../../5-workshop/5.9-cloudwatch-monitoring/)
- [Chi phí, bảo mật và clean-up](../../5-workshop/5.10-cost-security-cleanup/)
- [Kết quả và hướng phát triển](../../5-workshop/5.11-results-challenges-future/)
- [Bàn giao dự án](../../5-workshop/5.12-project-handover/)
