---
title: "Tuần 6 - Tích hợp phần cứng YOLO UNO"
date: "2026-07-06"
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

> **Thời gian:** 06/07/2026 – 12/07/2026
> **Vai trò:** Phụ trách phần cứng và firmware YOLO UNO; phối hợp với backend để kiểm tra telemetry, command và ACK.

## Mục tiêu

- Kết nối cảm biến và thiết bị chấp hành theo đúng firmware.
- Biên dịch, nạp và chạy firmware bằng PlatformIO.
- Xác minh luồng Wi-Fi, telemetry, command polling và ACK với backend.

## Công việc đã thực hiện

| Hạng mục | Công việc đã thực hiện | Kết quả/Bằng chứng |
| :--- | :--- | :--- |
| Đấu nối và an toàn nguồn | Đối chiếu GPIO trong `hardware/src/main.cpp`, nối DHT20, cảm biến ánh sáng, quạt, đèn/relay, servo, LCD1602 và kiểm tra mass/nguồn | Bảng đấu nối bám theo firmware; quạt và servo không lấy dòng trực tiếp từ GPIO |
| Môi trường firmware | Cấu hình PlatformIO, board ESP32-S3 và `secrets.h` cục bộ | Wi-Fi credential và backend URL không được đưa vào Git |
| Cảm biến và hiển thị | Đọc DHT20, giá trị ánh sáng analog và cập nhật LCD1602 | Serial Monitor/LCD hiển thị được dữ liệu cảm biến và trạng thái |
| Điều khiển actuator | Kiểm tra quạt, đèn/relay và servo tại góc đóng 0°/mở 90° | Các thiết bị chấp hành phản hồi đúng logic firmware |
| Tích hợp REST API | Gửi telemetry theo chu kỳ, thăm dò command, thực thi và gửi ACK | YOLO UNO giao tiếp hai chiều với FastAPI qua HTTP |
| Build và bằng chứng | Biên dịch firmware, theo dõi Serial Monitor và ghi video demo | PlatformIO tạo `firmware.bin`; video ghi nhận mô hình phần cứng hoạt động |
## Kết quả tuần

- YOLO UNO gửi được telemetry của `room_01` và nhận command từ backend.
- Quạt, đèn và servo rèm phản ứng theo lệnh trực tiếp; chế độ Auto dùng các ngưỡng đã định nghĩa trong firmware.
- Cơ chế lưu `lastAck` và `pendingAck` giúp thử gửi lại ACK mà không lặp thao tác vật lý.

## Khó khăn và bài học

Nguồn cho thiết bị chấp hành và việc chống thực thi lặp quan trọng không kém kết nối mạng. Khi ACK thất bại, firmware chỉ được gửi lại ACK, không được chạy lại actuator.

## Liên kết Workshop

- [5.6 Tích hợp phần cứng](../../5-workshop/5.6-hardware-integration/)
- [5.8 Kiểm thử đầu cuối](../../5-workshop/5.8-end-to-end-testing/)
