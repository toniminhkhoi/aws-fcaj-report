---
title: "Tuần 5 - Xây dựng API telemetry và command"
date: "2026-06-29"
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

> **Thời gian:** 29/06/2026 – 05/07/2026
> **Vai trò:** Phối hợp với thành viên backend để đối chiếu API với yêu cầu phần cứng và hạ tầng.

## Mục tiêu

- Hoàn thiện API nhận và truy xuất telemetry.
- Xây dựng API tạo, thăm dò và xác nhận command.
- Kiểm tra dữ liệu được lưu đúng trong PostgreSQL.

## Công việc đã thực hiện

| Hạng mục | Công việc đã thực hiện | Kết quả/Bằng chứng |
| :--- | :--- | :--- |
| Chuẩn hóa dữ liệu | Đối chiếu Pydantic schema với payload camelCase của firmware và ánh xạ `deviceId`, `lightIntensity` vào data model | Contract dữ liệu thống nhất giữa firmware, FastAPI và PostgreSQL |
| Telemetry ingestion | Hoàn thiện và kiểm tra `POST /api/telemetry` với payload có `device_id` là `room_01` | Telemetry hợp lệ tạo bản ghi mới trong `telemetry_logs` |
| Latest và history | Hoàn thiện API lấy dữ liệu mới nhất và lịch sử theo `device_id` | Dashboard nhận được dữ liệu cho telemetry card và biểu đồ |
| Vòng đời command | Hoàn thiện API tạo command, lấy command chờ và ACK | Command được lưu ở `Pending` và cùng ID chuyển sang `Executed` sau ACK |
| Kiểm thử API/database | Rà OpenAPI, gửi request có kiểm soát bằng `curl` và đối chiếu bản ghi SQL | Route, JSON response và trạng thái database khớp nhau |
## Kết quả tuần

- Hoàn thiện luồng telemetry, latest, history, command polling và ACK.
- Xác minh vòng đời lệnh bằng cùng một command ID trong API và PostgreSQL.
- Ghi nhận hạn chế: backend chưa kiểm tra chặt enum command và endpoint ACK chưa xác minh command thuộc đúng thiết bị trong route.

## Khó khăn và bài học

Trạng thái `Pending` có thể tồn tại rất ngắn khi thiết bị thăm dò thường xuyên. Cần lưu phản hồi tạo command trước, sau đó truy vấn cùng ID sau ACK để có bằng chứng đầy đủ.

## Liên kết Workshop

- [5.3 Đặc tả API và luồng dữ liệu](../../5-workshop/5.3-architecture-and-service-design/)
- [5.5 Backend và cơ sở dữ liệu](../../5-workshop/5.5-backend-and-database/)
- [5.8 Kiểm thử và xác minh đầu cuối](../../5-workshop/5.8-end-to-end-testing/)
