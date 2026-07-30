---
title: "Tổng quan Workshop"
date: "2026-07-28"
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

## Bối cảnh và vấn đề

Một Smart Room cần lưu dữ liệu môi trường để theo dõi theo thời gian, hỗ trợ điều khiển thiết bị từ xa và xác nhận từng lệnh đã được thực thi. Nếu cảm biến, dashboard và thiết bị chấp hành hoạt động rời rạc, người vận hành sẽ khó truy vết toàn bộ luồng dữ liệu hoặc phân biệt giữa “API đã nhận lệnh” và “thiết bị vật lý đã thực hiện lệnh”.

Workshop này xây dựng một quy trình thống nhất cho Smart Room được định danh bằng `device_id=room_01`. YOLO UNO thu thập nhiệt độ, độ ẩm và giá trị ánh sáng analog; FastAPI trên Amazon EC2 xử lý yêu cầu; Amazon RDS for PostgreSQL lưu telemetry và trạng thái lệnh; dashboard React + Vite hiển thị dữ liệu và tạo lệnh điều khiển. YOLO UNO thăm dò lệnh đang chờ, thực thi và gửi ACK để cập nhật trạng thái.

## Đối tượng sử dụng và giải pháp đề xuất

| Đối tượng sử dụng | Nhu cầu | Giá trị từ Workshop |
| :--- | :--- | :--- |
| Học viên FCAJ / người học AWS | Triển khai và xác minh một hệ thống đầu cuối trên AWS | Quy trình có thể thực hiện lại, từ thiết lập hạ tầng đến ứng dụng, phần cứng, giám sát và bằng chứng |
| Người vận hành Smart Room | Xem dữ liệu hiện tại, lịch sử và điều khiển thiết bị trong phòng | Một dashboard để theo dõi telemetry và điều khiển chế độ, quạt, đèn, rèm |
| Người bảo trì / lập trình viên | Truy vết lỗi qua ứng dụng, cơ sở dữ liệu, mạng và phần cứng | Bằng chứng liên kết từ request trình duyệt, log FastAPI, truy vấn SQL, `systemd`, Serial Monitor và CloudWatch |
| Người đánh giá dự án / mentor FCAJ | Đánh giá mức độ phù hợp với AWS, chiều sâu triển khai và đóng góp cá nhân | Quyết định kiến trúc, tiêu chí đo được, liên kết bằng chứng và tài liệu bàn giao rõ ràng |

## Mức độ phù hợp với FCAJ và AWS

Workshop đáp ứng mục tiêu học tập của FCAJ khi kết hợp kiến trúc đám mây, vận hành Linux, mạng, bảo mật, cơ sở dữ liệu, phát triển full-stack, IoT vật lý, kiểm thử, giám sát, viết tài liệu và bàn giao trong một hệ thống có thể truy vết.

Mỗi dịch vụ AWS đảm nhiệm một vai trò cụ thể: Amazon EC2 và EBS chạy backend FastAPI; Amazon RDS for PostgreSQL lưu dữ liệu; VPC và Security Group kiểm soát kết nối mạng; IAM Role cho phép EC2 gửi dữ liệu giám sát mà không cần lưu AWS credential dài hạn; Amazon CloudWatch thu thập log, metric và trạng thái alarm.

## Mục tiêu kỹ thuật

1. Nhận telemetry từ YOLO UNO cho Smart Room được định danh bằng `device_id=room_01`.
2. Truy xuất bản ghi mới nhất và lịch sử telemetry theo thứ tự thời gian.
3. Hỗ trợ đủ `8` lệnh firmware để điều khiển chế độ, quạt, đèn và rèm.
4. Theo dõi quá trình hoàn tất lệnh qua trạng thái `Pending` → `Executed` và ACK.
5. Chạy FastAPI dưới dạng dịch vụ `systemd`, đồng thời giám sát backend, EC2 và RDS bằng CloudWatch.
6. Bàn giao Workshop song ngữ, bằng chứng kiểm thử và tài liệu dự án có thể sử dụng lại.

## Phạm vi hiện tại

| Thành phần | Phạm vi đã triển khai |
| :--- | :--- |
| Định danh Smart Room | Một phòng được định danh bằng `device_id=room_01` |
| Cảm biến | DHT20 đo nhiệt độ/độ ẩm và cảm biến ánh sáng analog thô |
| Thiết bị chấp hành và hiển thị | Quạt, đèn/relay, servo rèm và màn hình trạng thái LCD1602 |
| Phần mềm thiết bị | Firmware PlatformIO cho YOLO UNO, hỗ trợ chế độ Auto/Manual và `8` lệnh |
| Backend và dữ liệu | FastAPI trên Amazon EC2; telemetry và lệnh được lưu trong Amazon RDS for PostgreSQL |
| Frontend | Dashboard React + Vite chạy trên máy người dùng |
| Giao tiếp | REST qua HTTP, thăm dò lệnh, thực thi và gửi ACK |
| Giám sát | Log backend, bốn widget metric EC2/RDS và năm cấu hình CloudWatch alarm |

## Yêu cầu chức năng

| Chức năng | Kết quả quan sát được |
| :--- | :--- |
| Nhận telemetry | Request hợp lệ tạo một bản ghi telemetry có thể xác định trong PostgreSQL |
| Telemetry mới nhất | Trả về bản ghi mới nhất của `device_id=room_01` |
| Lịch sử telemetry | Trả về dữ liệu của `device_id=room_01` theo thứ tự thời gian |
| Điều khiển quạt | Nhận và thực thi `FAN_ON`, `FAN_OFF` |
| Điều khiển đèn | Nhận và thực thi `LIGHT_ON`, `LIGHT_OFF` |
| Điều khiển rèm | Nhận và thực thi `CURTAIN_OPEN`, `CURTAIN_CLOSE` |
| Chế độ vận hành | `MODE_AUTO` bật điều khiển theo ngưỡng; `MODE_MANUAL` cho phép điều khiển trực tiếp |
| Vòng đời lệnh | Lệnh mới có trạng thái `Pending`; ACK thành công chuyển trạng thái thành `Executed` |
| Giám sát CloudWatch | Log và metric đã cấu hình được hiển thị; alarm đánh giá các ngưỡng tương ứng |

Dự án sử dụng logic dựa trên luật, không phải mô hình AI. Trong chế độ Auto, firmware điều khiển quạt khi `temperature >= 30 °C`, đèn khi giá trị analog thô `< 350` và rèm quanh ngưỡng `< 700`. Khi nhận lệnh điều khiển trực tiếp, firmware chuyển sang chế độ Manual.

## Đầu ra cụ thể

| Đầu ra | Sản phẩm hoặc bằng chứng |
| :--- | :--- |
| Hạ tầng AWS | Bằng chứng cấu hình EC2/EBS, RDS, VPC/subnet, Security Group, IAM Role và CloudWatch |
| Backend đang chạy | Dịch vụ `aws-iot-backend` ở trạng thái hoạt động và `GET /api/health` trả HTTP 200 |
| Lưu trữ PostgreSQL | Bằng chứng bảng và truy vấn cho `devices`, `telemetry_logs`, `commands` |
| Tích hợp YOLO UNO | Tài liệu GPIO/nối dây, kết quả build PlatformIO, luồng telemetry, thực thi lệnh và ACK |
| Dashboard | Dữ liệu mới nhất/lịch sử, thao tác điều khiển, trạng thái lệnh, phần phân tích đề xuất và demo điều khiển phần cứng |
| Giám sát | Log backend, bốn widget metric EC2/RDS và năm cấu hình CloudWatch alarm |
| Kiểm thử | Ma trận T01–T12 với toàn bộ kết quả được ghi nhận là Đạt và có bằng chứng liên kết |
| Bàn giao | Repository mã nguồn, README và Workshop song ngữ, liên kết tham khảo và checklist bàn giao cuối cùng |

## Tiêu chí thành công đo được

| ID | Tiêu chí | Cách đo | Trạng thái |
| :--- | :--- | :--- | :--- |
| S01 | Backend sẵn sàng | `GET /api/health` trả HTTP 200 từ dịch vụ đã triển khai | Đạt |
| S02 | Lưu telemetry | Một request POST hợp lệ cho `device_id=room_01` tạo một dòng có thể xác định trong `telemetry_logs` | Đạt |
| S03 | Truy xuất dữ liệu | Request mới nhất và lịch sử trả đúng dữ liệu `room_01` đã lưu theo thứ tự mong đợi | Đạt |
| S04 | Điều khiển vật lý | Sáu lệnh điều khiển trực tiếp được xác minh bằng dữ liệu lệnh và bằng chứng vật lý | Đạt |
| S05 | Điều khiển chế độ | `MODE_AUTO` và `MODE_MANUAL` được xác minh qua hành vi firmware | Đạt |
| S06 | Hoàn tất lệnh | Cùng một ID lệnh được ghi nhận ở `Pending`, sau đó chuyển sang `Executed` sau ACK | Đạt |
| S07 | Giám sát | Log backend, bốn widget metric EC2/RDS và năm cấu hình alarm đều hiển thị | Đạt |
| S08 | Khả năng tái tạo và an toàn | T01–T12 đều được ghi nhận là Đạt, tài liệu bàn giao đã có và credential không xuất hiện trong repository | Đạt |

Bằng chứng cho các tiêu chí trên được trình bày tại mục 5.5–5.9 và 5.12.

## Gợi ý xử lý sự cố

Nếu chưa xác định được thành phần gây lỗi, hãy truy vết một request qua thẻ Network của trình duyệt, log FastAPI, bản ghi PostgreSQL, Serial Monitor của thiết bị và trạng thái ACK. Chỉ đánh dấu Đạt sau khi đã ghi nhận bằng chứng.

Tiếp theo: [chuẩn bị tài khoản, công cụ và phần cứng](../5.2-Prerequisites/).