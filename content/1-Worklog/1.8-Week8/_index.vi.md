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
- Dùng thao tác dashboard làm điểm bắt đầu, sau đó đối chiếu cùng dữ liệu hoặc command ID qua DevTools/API, RDS và firmware.
- Thiết lập log, metric và alarm cho ALB/ASG/EC2/RDS.
- Hoàn thiện rà soát bảo mật, tài liệu song ngữ, video demo và nội dung bàn giao.

## Công việc đã thực hiện

| Hạng mục | Công việc đã thực hiện | Kết quả/Bằng chứng |
| :--- | :--- | :--- |
| Chuẩn bị kiểm thử | Ghi phiên bản mã nguồn, firmware, Region AWS và điều kiện kiểm thử; xác nhận health check | Có baseline cấu hình để tái tạo và đối chiếu kết quả |
| Kiểm thử telemetry | Gửi telemetry từ YOLO UNO, kiểm tra latest/history, PostgreSQL và dashboard; thực hiện thêm một request `curl` có kiểm soát | Dữ liệu `room_01` nhất quán; ảnh API/database chứng minh riêng luồng FastAPI → RDS |
| Kiểm thử command và actuator | Tạo command từ dashboard, lấy ID qua phản hồi API/DevTools, theo dõi từ `Pending` đến `Executed` trong backend/RDS và kiểm tra quạt, đèn, rèm | Ma trận kiểm thử command/ACK đạt; video ghi lại phản ứng vật lý của actuator |
| AWS edge và tính sẵn sàng | Bổ sung bằng chứng CloudFront/WAF/S3 private, ALB/target group/ASG, EBS mã hóa và RDS Multi-AZ/backup | Hai backend Healthy, route browser/thiết bị ổn định và bằng chứng primary/standby |
| CloudWatch | Cấu hình CloudWatch Agent, backend log, dashboard `ec2-rds-metrics` và tám alarm ALB/ASG/EC2/RDS | Có log tập trung, metric vận hành và kết quả đánh giá alarm; trạng thái thiếu dữ liệu được ghi chú |
| Rà soát vận hành | Kiểm tra IAM Role, Security Group, RDS public access, secrets, chi phí và các giới hạn hiện tại | Checklist bảo mật/chi phí và danh sách điểm cần tiếp tục hoàn thiện |
| Tài liệu và bàn giao | Rà Workshop Anh–Việt, README, caption, test matrix, ảnh, video và checklist | Gói tài liệu cuối khớp với hệ thống đã triển khai và bằng chứng đã thu thập |
## Kết quả trong tuần

- Các ca kiểm thử telemetry, history, command, polling, actuator và ACK trong ma trận đều đạt.
- Phân biệt rõ bằng chứng API/database với bằng chứng phần cứng.
- CloudWatch nhận backend log, hiển thị metric ALB/ASG/EC2/RDS và đánh giá tám alarm.
- Hoàn thiện báo cáo song ngữ, Workshop, README, hướng dẫn vận hành và nội dung bàn giao.
- Video demo chứng minh thao tác dashboard và phản ứng vật lý của thiết bị: [Xem video demo](https://drive.google.com/file/d/1T97dUY58hbT2ppxvg7ESR12Jg9BA828W/view?usp=sharing).

## Khó khăn, cách xử lý và bài học

Trạng thái `Pending` tồn tại ngắn khi thiết bị polling nhanh, nên nhóm theo dõi cùng command ID trước và sau ACK. Với CloudWatch, các alarm `Insufficient data` được ghi nhận là thiếu bằng chứng metric thay vì xem như trạng thái đạt; agent, namespace, dimension và IAM là các điểm cần kiểm tra khi xử lý. Em rút ra rằng một kết quả Pass phải gắn với tiêu chí, kết quả thực tế và bằng chứng; tài liệu cũng phải phân biệt rõ phần đã triển khai với hướng phát triển.

## Giới hạn hiện tại

CloudFront cung cấp viewer HTTPS, nhưng luồng ALB/thiết bị dùng HTTP; API authentication, WAF Block, notification action và diễn tập failover có kiểm soát chưa hoàn tất. Các giới hạn này được ghi nhận để tiếp tục cải tiến.

## Liên kết Workshop

- [Kiểm thử end-to-end](../../5-workshop/5.8-end-to-end-testing/)
- [Giám sát bằng CloudWatch](../../5-workshop/5.9-cloudwatch-monitoring/)
- [Chi phí, bảo mật và clean-up](../../5-workshop/5.10-cost-security-cleanup/)
- [Kết quả và hướng phát triển](../../5-workshop/5.11-results-challenges-future/)
- [Bàn giao dự án](../../5-workshop/5.12-project-handover/)
