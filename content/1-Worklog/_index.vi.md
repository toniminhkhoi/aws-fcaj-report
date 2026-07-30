---
title: "Nhật ký công việc"
date: "2026-06-01"
weight: 1
chapter: false
pre: " <b> 1. </b> "
---

Nhật ký này ghi lại quá trình triển khai dự án **AWS IoT Monitoring and Control Dashboard** trong kỳ thực tập từ **01/06/2026 đến 31/07/2026**. Nội dung bám sát timeline trong Proposal và bằng chứng kỹ thuật của Workshop.

Khoảng thời gian trên tương đương 8 tuần và 5 ngày. Nhật ký được tổ chức thành 8 giai đoạn: bảy tuần đầu kéo dài 7 ngày; Tuần 8 từ 20/07 đến 31/07 bao gồm các ngày tích hợp, giám sát, hoàn thiện tài liệu và bàn giao còn lại. Cách chia này bao quát toàn bộ kỳ thực tập mà không tạo thêm Tuần 9.

Đây là dự án nhóm. Mỗi trang tuần phân biệt phần tôi phụ trách với vai trò **AWS và Hardware Lead** và những nội dung phối hợp cùng các thành viên backend, frontend và tài liệu.

| Tuần | Thời gian | Mốc triển khai | Kết quả chính |
| :---: | :--- | :------------- | :------------ |
| 1 | 01/06–07/06 | [Phân tích yêu cầu và lập kế hoạch](1.1-week1/) | Chốt bài toán, phạm vi `room_01`, phân công và kiến trúc ban đầu |
| 2 | 08/06–14/06 | [Thiết kế kiến trúc AWS và nền tảng mạng](1.2-week2/) | Hoàn thiện thiết kế VPC, subnet, Security Group, IAM và luồng dữ liệu |
| 3 | 15/06–21/06 | [Triển khai Amazon EC2 và Amazon RDS](1.3-week3/) | EC2 hoạt động, RDS PostgreSQL sẵn sàng và kết nối mạng được xác minh |
| 4 | 22/06–28/06 | [Xây dựng nền tảng backend và cơ sở dữ liệu](1.4-week4/) | FastAPI chạy bằng `systemd`, kết nối RDS và có các bảng ứng dụng |
| 5 | 29/06–05/07 | [Xây dựng API telemetry và command](1.5-week5/) | Hoàn thiện telemetry, latest, history, command polling và ACK |
| 6 | 06/07–12/07 | [Tích hợp phần cứng YOLO UNO](1.6-week6/) | Đọc cảm biến, điều khiển actuator, kết nối Wi-Fi, gửi telemetry và xử lý command |
| 7 | 13/07–19/07 | [Phát triển frontend dashboard](1.7-week7/) | Hiển thị telemetry/lịch sử và tạo command có thể theo dõi |
| 8 | 20/07–31/07 | [Tích hợp, kiểm thử, CloudWatch và bàn giao](1.8-week8/) | Xác minh end-to-end, hoàn thiện giám sát, rà soát bảo mật, tài liệu và video demo |

> Nhật ký phản ánh đúng mô hình hiện tại: một EC2 chạy FastAPI, một RDS for PostgreSQL, dashboard React chạy phía người dùng và một thiết bị mẫu `room_01`. Dự án không tuyên bố đã triển khai Auto Scaling, SQS, kiến trúc event-driven hoặc BMS đa chi nhánh.
