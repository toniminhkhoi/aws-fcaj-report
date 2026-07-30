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

| Thời gian | Công việc | Kết quả ghi nhận |
| :--- | :--- | :--- |
| 06/07 | Đối chiếu GPIO trong `hardware/src/main.cpp` và nối DHT20, cảm biến ánh sáng, quạt, đèn/relay, servo và LCD1602 | Sơ đồ đấu nối lấy firmware đang chạy làm nguồn chuẩn; servo dùng D11/GPIO38 |
| 07/07 | Kiểm tra nguồn, nối chung mass và trạng thái an toàn của thiết bị chấp hành | Không cấp dòng trực tiếp cho quạt hoặc servo từ GPIO |
| 08/07 | Cấu hình PlatformIO, board ESP32-S3 và file `secrets.h` cục bộ | Wi-Fi, URL backend và `room_01` không bị đưa vào Git |
| 09/07 | Đọc DHT20 và giá trị ánh sáng analog; kiểm tra quạt, đèn, servo 0°/90° và LCD | Cảm biến và thiết bị chấp hành phản hồi theo logic firmware |
| 10–11/07 | Gửi telemetry mỗi 5 giây; thăm dò command mỗi 2 giây; thực thi lệnh và gửi ACK | YOLO UNO giao tiếp với FastAPI qua HTTP và xử lý tám command được hỗ trợ |
| 12/07 | Biên dịch firmware, theo dõi Serial Monitor và ghi lại video demo | PlatformIO tạo `firmware.bin`; video thể hiện mô hình phần cứng hoạt động |

## Kết quả tuần

- YOLO UNO gửi được telemetry của `room_01` và nhận command từ backend.
- Quạt, đèn và servo rèm phản ứng theo lệnh trực tiếp; chế độ Auto dùng các ngưỡng đã định nghĩa trong firmware.
- Cơ chế lưu `lastAck` và `pendingAck` giúp thử gửi lại ACK mà không lặp thao tác vật lý.

## Khó khăn và bài học

Nguồn cho thiết bị chấp hành và việc chống thực thi lặp quan trọng không kém kết nối mạng. Khi ACK thất bại, firmware chỉ được gửi lại ACK, không được chạy lại actuator.

## Liên kết Workshop

- [5.6 Tích hợp phần cứng](../../5-workshop/5.6-hardware-integration/)
- [5.8 Kiểm thử đầu cuối](../../5-workshop/5.8-end-to-end-testing/)
