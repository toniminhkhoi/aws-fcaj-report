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

| Thời gian | Công việc | Kết quả ghi nhận |
| :--- | :--- | :--- |
| 29–30/06 | Đối chiếu schema Pydantic cho telemetry với payload camelCase của firmware | Backend nhận `deviceId`, `lightIntensity` và ánh xạ sang mô hình dữ liệu |
| 01/07 | Hoàn thiện `POST /api/telemetry` | Telemetry hợp lệ của `room_01` tạo bản ghi trong `telemetry_logs` |
| 02/07 | Hoàn thiện API dữ liệu mới nhất và lịch sử | Dashboard có thể gọi `/latest` và `/history` theo `device_id` |
| 03–04/07 | Hoàn thiện API tạo command, lấy command chờ và ACK | Command mới được lưu ở `Pending`; ACK chuyển đúng ID sang `Executed` |
| 05/07 | Kiểm tra OpenAPI, chạy yêu cầu `curl` và đối chiếu bằng truy vấn SQL | Các route, phản hồi JSON và trạng thái cơ sở dữ liệu khớp nhau |

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
