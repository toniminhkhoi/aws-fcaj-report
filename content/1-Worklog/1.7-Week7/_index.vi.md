---
title: "Tuần 7 - Phát triển frontend dashboard"
date: "2026-07-13"
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---

> **Thời gian:** 13/07/2026 – 19/07/2026
> **Vai trò:** Hỗ trợ kết nối frontend với endpoint EC2; phối hợp kiểm tra dashboard và luồng điều khiển.

## Mục tiêu

- Hiển thị telemetry mới nhất, lịch sử và trạng thái backend.
- Tạo bảng điều khiển quạt, đèn và rèm.
- Gửi command từ dashboard, kiểm tra phản hồi backend và xác định phần còn thiếu trong việc theo dõi ACK.

## Công việc đã thực hiện

| Hạng mục | Công việc đã thực hiện | Kết quả/Bằng chứng |
| :--- | :--- | :--- |
| Chuẩn bị frontend | Thiết lập React, Vite, TypeScript, Tailwind CSS và các dependencies cần thiết | Dashboard chạy ổn định trên môi trường cục bộ |
| Cấu hình kết nối API | Dùng đường dẫn tương đối `/api`; Vite proxy trỏ đến ALB trong môi trường phát triển, còn CloudFront behavior `/api/*` xử lý route production | Các component không hard-code backend URL; đường đi development và production được phân biệt rõ |
| Hiển thị telemetry | Kết nối health, latest và history; xây dựng telemetry card và biểu đồ | Dashboard hiển thị dữ liệu mới nhất, lịch sử và trạng thái kết nối |
| Điều khiển thiết bị | Xây dựng nút quạt, đèn, rèm và chế độ Auto/Manual; kiểm tra request tạo command | UI gửi command đến backend, nhưng chưa lưu command ID hoặc theo dõi trạng thái `Pending`/`Executed` |
| Rà dữ liệu và nhãn | Kiểm tra fallback mô phỏng và cách mô tả giá trị ánh sáng | Xác định fallback chưa có nhãn rõ ràng và dữ liệu ánh sáng thô vẫn được hiển thị với đơn vị Lux; ghi nhận để khắc phục |
| Tích hợp và sửa lỗi | Dùng DevTools Network kiểm tra route, payload, response, request trùng và hành vi khi API lỗi | Xác minh route production hoạt động; phát hiện cơ chế fallback có thể trả kết quả thành công giả khi backend không phản hồi |
## Kết quả tuần

- Dashboard hiển thị dữ liệu mới nhất và lịch sử của `room_01`.
- Bảng điều khiển tạo command khi API khả dụng; command ID và trạng thái ACK phải được đối chiếu bằng DevTools/API/PostgreSQL thay vì trực tiếp trên UI.
- Ghi nhận các hạn chế cần tiếp tục xử lý: chế độ trên UI còn mang tính cục bộ, fallback mô phỏng chưa gắn nhãn và thao tác lỗi vẫn có thể hiển thị thành công.

## Khó khăn và bài học

Phản hồi HTTP thành công chỉ chứng minh backend đã nhận command, chưa chứng minh thiết bị vật lý đã thực thi. Giao diện phải chờ ACK/`Executed` hoặc chỉ rõ trạng thái đang chờ.

## Liên kết Workshop

- [5.7 Tích hợp frontend](../../5-workshop/5.7-frontend-integration/)
- [5.8 Kiểm thử đầu cuối](../../5-workshop/5.8-end-to-end-testing/)
