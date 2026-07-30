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

| Thời gian | Công việc | Kết quả ghi nhận |
| :--- | :--- | :--- |
| 13/07 | Chuẩn bị dự án React + Vite + TypeScript + Tailwind CSS và cài dependencies | Dashboard chạy trên máy cục bộ |
| 14/07 | Cấu hình Vite proxy cho đường dẫn tương đối `/api` | URL EC2 được tập trung tại cấu hình thay vì lặp trong nhiều component |
| 15/07 | Kết nối `/latest`, `/history` và health check của backend | Telemetry card, biểu đồ lịch sử và trạng thái kết nối dùng dữ liệu API |
| 16–17/07 | Xây dựng nút điều khiển quạt, đèn, rèm và chế độ Auto/Manual | UI gửi đúng command và hiển thị ID/trạng thái do backend trả về |
| 18/07 | Rà soát nguồn dữ liệu thật/mô phỏng và nhãn ánh sáng | Dữ liệu mô phỏng được phân biệt; giá trị ADC không được khẳng định là Lux |
| 19/07 | Dùng DevTools Network để kiểm tra route, payload, phản hồi và lỗi | Có checklist cho yêu cầu trùng, trạng thái `Pending` và lỗi backend |

## Kết quả tuần

- Dashboard hiển thị dữ liệu mới nhất và lịch sử của `room_01`.
- Bảng điều khiển tạo command có thể truy vết qua backend.
- Ghi nhận các hạn chế cần tiếp tục xử lý: chế độ trên UI còn mang tính cục bộ và không được dùng dữ liệu mô phỏng làm bằng chứng vận hành.

## Khó khăn và bài học

Phản hồi HTTP thành công chỉ chứng minh backend đã nhận command, chưa chứng minh thiết bị vật lý đã thực thi. Giao diện phải chờ ACK/`Executed` hoặc chỉ rõ trạng thái đang chờ.

## Liên kết Workshop

- [5.7 Tích hợp frontend](../../5-workshop/5.7-frontend-integration/)
- [5.8 Kiểm thử đầu cuối](../../5-workshop/5.8-end-to-end-testing/)
