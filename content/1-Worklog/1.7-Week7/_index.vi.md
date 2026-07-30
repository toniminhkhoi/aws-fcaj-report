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
- Theo dõi command từ lúc gửi đến khi backend ghi nhận ACK.

## Công việc đã thực hiện

| Hạng mục | Công việc đã thực hiện | Kết quả/Bằng chứng |
| :--- | :--- | :--- |
| Chuẩn bị frontend | Thiết lập React, Vite, TypeScript, Tailwind CSS và các dependencies cần thiết | Dashboard chạy ổn định trên môi trường cục bộ |
| Cấu hình kết nối API | Dùng Vite proxy cho đường dẫn tương đối `/api` và tập trung backend URL trong cấu hình | Frontend kết nối EC2 mà không lặp URL trong nhiều component |
| Hiển thị telemetry | Kết nối health, latest và history; xây dựng telemetry card và biểu đồ | Dashboard hiển thị dữ liệu mới nhất, lịch sử và trạng thái kết nối |
| Điều khiển thiết bị | Xây dựng nút quạt, đèn, rèm và chế độ Auto/Manual; hiển thị command ID/trạng thái | UI gửi đúng command và cho phép theo dõi phản hồi backend |
| Rà dữ liệu và nhãn | Phân biệt dữ liệu thật/mô phỏng và kiểm tra cách mô tả giá trị ánh sáng | Dữ liệu mô phỏng được nhận diện; giá trị ADC không bị ghi nhầm thành Lux |
| Tích hợp và sửa lỗi | Dùng DevTools Network kiểm tra route, payload, response, request trùng và trạng thái `Pending` | Checklist tích hợp và các lỗi frontend–backend được xử lý trước kiểm thử end-to-end |
## Kết quả tuần

- Dashboard hiển thị dữ liệu mới nhất và lịch sử của `room_01`.
- Bảng điều khiển tạo command có thể truy vết qua backend.
- Ghi nhận các hạn chế cần tiếp tục xử lý: chế độ trên UI còn mang tính cục bộ và không được dùng dữ liệu mô phỏng làm bằng chứng vận hành.

## Khó khăn và bài học

Phản hồi HTTP thành công chỉ chứng minh backend đã nhận command, chưa chứng minh thiết bị vật lý đã thực thi. Giao diện phải chờ ACK/`Executed` hoặc chỉ rõ trạng thái đang chờ.

## Liên kết Workshop

- [5.7 Tích hợp frontend](../../5-workshop/5.7-frontend-integration/)
- [5.8 Kiểm thử đầu cuối](../../5-workshop/5.8-end-to-end-testing/)
