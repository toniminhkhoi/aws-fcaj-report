---
title: "Video Demo"
date: "2026-07-30"
weight: 2
chapter: false
pre: " <b> 8.2. </b> "
---

Video trình diễn hoạt động end-to-end của hệ thống **AWS IoT Monitoring and Control Dashboard**.

## Nội dung demo

Video thể hiện:

1. React + Vite dashboard nhận telemetry gần thời gian thực.
2. FastAPI backend hoạt động trên Amazon EC2.
3. Telemetry và command được lưu trong Amazon RDS for PostgreSQL.
4. YOLO UNO thăm dò command từ backend.
5. Quạt, đèn và servo rèm phản ứng với lệnh điều khiển.
6. Thiết bị gửi ACK sau khi thực thi.
7. Backend cập nhật command sang trạng thái `Executed`.

Video không bao gồm phần Amazon CloudWatch. Bằng chứng về log, metric và alarm được trình bày bằng ảnh chụp tại [mục 5.9 - Giám sát bằng CloudWatch]({{% relref "5-Workshop/5.9-CloudWatch-Monitoring/_index.vi.md" %}}).

## Liên kết video

[▶ Xem video demo end-to-end](https://drive.google.com/file/d/1T97dUY58hbT2ppxvg7ESR12Jg9BA828W/view?usp=sharing)
