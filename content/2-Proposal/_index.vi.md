---
title: "Bản đề xuất"
date: "2026-06-15"
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# AWS IoT Monitoring and Control Dashboard

### Bản tải xuống: <a href="/files/2-Proposal/IoT_Dashboard_Proposal_vi.pdf" download>IoT Dashboard Proposal (PDF)</a>

> **Cần cập nhật PDF:** file PDF hiện được giữ để tham khảo và phải được xuất lại theo Proposal mới trước khi nộp bài.

---

## 1. Tóm tắt dự án

**AWS IoT Monitoring and Control Dashboard** giải quyết nhu cầu theo dõi điều kiện môi trường và điều khiển thiết bị trong phòng từ một giao diện tập trung. Hệ thống kết nối phần cứng YOLO UNO với backend triển khai trên AWS, cho phép người dùng xem nhiệt độ, độ ẩm và ánh sáng, gửi lệnh điều khiển từ xa, đồng thời xác minh thiết bị vật lý đã thực thi thông qua cơ chế xác nhận lệnh.

Trong mô hình Workshop hiện tại:

- YOLO UNO thu thập telemetry môi trường và điều khiển quạt, đèn, servo rèm;
- FastAPI chạy trên Amazon EC2 dưới sự quản lý của `aws-iot-backend.service`;
- Amazon RDS for PostgreSQL lưu telemetry và trạng thái lệnh;
- frontend React + Vite chạy cục bộ, hiển thị dữ liệu gần thời gian thực, lịch sử, điều khiển và đề xuất dựa trên luật;
- thiết bị thăm dò lệnh và gửi ACK sau khi thực thi; và
- Amazon CloudWatch thu thập log backend, metric vận hành và trạng thái alarm.

Kết quả là một mô hình Smart Room có thể tái tạo, thể hiện trọn vẹn luồng từ cảm biến vật lý đến lưu trữ trên cloud, hiển thị dashboard, điều khiển từ xa và xác nhận từ thiết bị.

---

## 2. Bài toán và người dùng mục tiêu

### 2.1 Bài toán

Trong phòng học, phòng lab hoặc phòng thiết bị nhỏ, cảm biến và thiết bị chấp hành thường hoạt động rời rạc. Giá trị cảm biến có thể chỉ được xem tại thiết bị, dữ liệu lịch sử không được lưu tập trung, còn người vận hành khó theo dõi điều kiện trong phòng hoặc điều khiển thiết bị từ xa. Một thao tác trên dashboard cũng chưa đủ để chứng minh quạt, đèn hoặc rèm đã thực sự hoàn thành lệnh.

Việc xử lý sự cố thiếu tính liên kết khi log ứng dụng, metric hạ tầng, bản ghi cơ sở dữ liệu và trạng thái thiết bị nằm ở nhiều nơi. Dự án tạo một quy trình có thể truy vết cho telemetry, lệnh điều khiển, ACK và giám sát.

### 2.2 Người dùng mục tiêu

| Đối tượng | Nhu cầu chính |
|---|---|
| Người quản lý phòng hoặc phòng lab | Theo dõi điều kiện môi trường và trạng thái thiết bị chấp hành trên một giao diện |
| Giảng viên và sinh viên | Quan sát telemetry, dữ liệu lịch sử và thử nghiệm luồng IoT đầu cuối |
| Người vận hành | Điều khiển quạt, đèn và rèm từ xa |
| Nhóm phát triển | Gỡ lỗi thông qua log, metric, API, bản ghi cơ sở dữ liệu và trạng thái lệnh |

### 2.3 Giá trị mang lại

- Dashboard tập trung cho dữ liệu hiện tại, lịch sử telemetry và điều khiển.
- Giao tiếp hai chiều giữa backend trên cloud và phần cứng vật lý.
- Lưu lịch sử telemetry và lệnh trong PostgreSQL.
- Xác minh hành động vật lý qua trạng thái `Pending` và `Executed`.
- Tập trung log và metric để hỗ trợ xử lý sự cố.
- Nền tảng có tài liệu rõ ràng để phát triển thêm phòng hoặc thiết bị khi cần.

---

## 3. Mục tiêu và phạm vi

### 3.1 Mục tiêu phần cứng

- Đọc nhiệt độ, độ ẩm từ DHT20 và giá trị cảm biến ánh sáng analog.
- Điều khiển quạt, đèn/relay, servo rèm; hiển thị trạng thái trên LCD1602 I2C.
- Kết nối Wi-Fi và giao tiếp qua HTTP REST.
- Hỗ trợ 8 lệnh: `MODE_AUTO`, `MODE_MANUAL`, `FAN_ON`, `FAN_OFF`, `LIGHT_ON`, `LIGHT_OFF`, `CURTAIN_OPEN` và `CURTAIN_CLOSE`.

### 3.2 Mục tiêu backend và cloud

- Triển khai FastAPI, Uvicorn, SQLAlchemy và Pydantic trên Amazon EC2.
- Quản lý backend bằng `aws-iot-backend.service`.
- Lưu telemetry và lệnh trong database `iot_dashboard` trên RDS PostgreSQL.
- Cấu hình VPC, định tuyến và Security Group phù hợp.
- Giữ RDS không public, chỉ nhận PostgreSQL từ Security Group của EC2.
- Dùng IAM Role/Instance Profile cho quyền của CloudWatch Agent.

### 3.3 Mục tiêu frontend

- Hiển thị nhiệt độ, độ ẩm, ánh sáng và biểu đồ lịch sử bằng REST polling.
- Tạo lệnh cho quạt, đèn, rèm, chế độ Manual và Auto.
- Trình bày phân tích dựa trên luật/ngưỡng, không mô tả là machine learning.

### 3.4 Mục tiêu giám sát và tài liệu

- Gửi log backend lên CloudWatch Logs.
- Theo dõi CPU, bộ nhớ, đĩa EC2; CPU và số kết nối RDS.
- Cấu hình alarm cho các ngưỡng vận hành quan trọng.
- Bàn giao Workshop song ngữ, source code, kiến trúc, video demo và hướng dẫn clean-up.

### 3.5 Phạm vi hiện tại

Mô hình dùng `room_01` làm mã định danh logic cho phòng mẫu. Phạm vi gồm ba loại dữ liệu môi trường, ba thiết bị chấp hành, HTTP REST, polling lệnh, ACK, một EC2, một RDS PostgreSQL Single-AZ, frontend chạy cục bộ và CloudWatch.

### 3.6 Ngoài phạm vi

Phiên bản hiện tại chưa bao gồm triển khai nhiều phòng ở môi trường vận hành thực tế, xác thực người dùng, HTTPS với tên miền riêng, tự động mở rộng theo tải, khôi phục thảm họa liên vùng sẵn sàng, ứng dụng di động, quy trình triển khai tự động hoặc mô hình machine learning đã huấn luyện. Các khả năng này cần được đánh giá riêng về yêu cầu, chi phí, bảo mật và độ phức tạp vận hành.

---

## 4. Kiến trúc giải pháp

![Kiến trúc AWS IoT Monitoring and Control Dashboard](/images/2-Proposal/IoT_Dashboard_Architecture.png)

*Kiến trúc triển khai hiện tại của AWS IoT Monitoring and Control Dashboard, gồm frontend chạy cục bộ, YOLO UNO, FastAPI trên EC2, RDS PostgreSQL và Amazon CloudWatch.*

### 4.1 Vị trí tài nguyên

| Vị trí | Thành phần | Trách nhiệm |
|---|---|---|
| Ngoài AWS | Người dùng, frontend React + Vite cục bộ, YOLO UNO ESP32-S3 | Tương tác, telemetry, thực thi lệnh và ACK |
| AWS Cloud, ngoài VPC | IAM Role/Instance Profile của EC2, CloudWatch | Quyền, log, metric, dashboard và alarm |
| Amazon VPC | Internet Gateway, public route table, public application subnet, private DB subnet | Ranh giới mạng và định tuyến |
| Public application subnet | Amazon EC2 | FastAPI backend và CloudWatch Agent |
| Private DB subnet qua DB Subnet Group | Amazon RDS for PostgreSQL | Lưu riêng tư telemetry và lệnh |
| Gắn với EC2 trong cùng AZ | EBS `gp3` 10 GiB làm root volume | Hệ điều hành, ứng dụng và file runtime |

CloudWatch Agent chạy trong EC2 để gửi log backend cùng metric bộ nhớ và đĩa lên CloudWatch. RDS tự xuất metric do dịch vụ quản lý. EBS là root volume gắn với EC2, không phải một dịch vụ độc lập nằm trong subnet.

### 4.2 Luồng telemetry

1. YOLO UNO đọc DHT20 và cảm biến ánh sáng analog.
2. Firmware gửi `POST /api/telemetry` đến FastAPI trên EC2.
3. FastAPI kiểm tra rồi lưu bản ghi vào RDS PostgreSQL.
4. Frontend định kỳ gọi endpoint latest và history.
5. Dashboard hiển thị dữ liệu hiện tại và lịch sử.

### 4.3 Luồng lệnh và xác nhận

1. Người vận hành tạo lệnh từ frontend.
2. FastAPI lưu lệnh ở trạng thái `Pending`.
3. YOLO UNO thăm dò lệnh mới nhất của `room_01`.
4. Firmware thực thi lệnh cho thiết bị chấp hành hoặc chế độ vận hành.
5. YOLO UNO gửi ACK kèm command ID dạng số.
6. FastAPI cập nhật đúng bản ghi sang `Executed`.

### 4.4 Các REST endpoint chính

| Phương thức | Endpoint | Mục đích |
|---|---|---|
| `GET` | `/api/health` | Kiểm tra backend |
| `POST` | `/api/telemetry` | Lưu telemetry |
| `GET` | `/api/devices/{device_id}/latest` | Lấy telemetry mới nhất |
| `GET` | `/api/devices/{device_id}/history` | Lấy lịch sử telemetry |
| `POST` | `/api/devices/{device_id}/commands` | Tạo lệnh |
| `GET` | `/api/devices/{device_id}/commands/latest` | Thăm dò lệnh mới nhất |
| `POST` | `/api/devices/{device_id}/commands/{command_id}/ack` | Xác nhận thực thi |

### 4.5 Ranh giới bảo mật

- RDS không cho phép truy cập trực tiếp từ Internet.
- Cổng TCP `5432` của RDS chỉ nhận từ Security Group của EC2, không nhận public CIDR.
- EC2 dùng IAM Role/Instance Profile cho CloudWatch thay vì AWS credential tĩnh.
- Giá trị runtime được cung cấp qua biến môi trường hoặc file secret cục bộ.
- Không commit `backend/.env`, `hardware/include/secrets.h`, `*.pem`, mật khẩu hoặc credential.

---

## 5. Kế hoạch triển khai kỹ thuật

### Giai đoạn 1 — Yêu cầu và nền tảng AWS

Thống nhất yêu cầu chức năng và tiêu chí nghiệm thu của Smart Room, phân công trách nhiệm cho từng thành viên, đồng thời quy ước `room_01` là mã định danh logic của phòng mẫu. Sau đó, nhóm thiết kế hạ tầng AWS, triển khai EC2 và RDS, rồi kiểm tra kết nối riêng tư từ backend đến cơ sở dữ liệu.

### Giai đoạn 2 — Backend và cơ sở dữ liệu

Thiết kế schema PostgreSQL; xây dựng health, telemetry, latest, history, tạo lệnh, polling và ACK; cấu hình biến môi trường và systemd.

### Giai đoạn 3 — Phần cứng và firmware

Kết nối DHT20, cảm biến ánh sáng, quạt, đèn/relay, servo và LCD; phát triển firmware PlatformIO cho cảm biến, Wi-Fi, telemetry, polling, điều khiển, chế độ và ACK; xác minh đủ 8 lệnh.

### Giai đoạn 4 — Frontend và tích hợp

Xây dựng dashboard React + Vite + TypeScript, biểu đồ, bảng điều khiển và đề xuất dựa trên luật; tích hợp toàn hệ thống và xử lý lỗi API, CORS, trạng thái lệnh, phần cứng.

### Giai đoạn 5 — Giám sát, kiểm thử và bàn giao

Cấu hình CloudWatch Agent, log, metric và 5 alarm; kiểm thử đầu cuối; rà soát bảo mật; hoàn thiện Workshop song ngữ, bằng chứng, demo, clean-up và bàn giao.

---

## 6. Timeline và các mốc triển khai

Proposal sử dụng đúng timeline 8 tuần của Worklog, từ **01/06/2026 đến 31/07/2026**.

| Tuần | Thời gian | Công việc chính | Mốc hoàn thành |
|---|---|---|---|
| **Tuần 1** | 01/06–07/06 | Yêu cầu, phạm vi, kiến trúc ban đầu, phân công và kế hoạch | Thống nhất chức năng, nghiệm thu, kiến trúc và vai trò |
| **Tuần 2** | 08/06–14/06 | Kiến trúc AWS, VPC, IAM và Security Group | Hoàn tất thiết kế cloud và bảo mật |
| **Tuần 3** | 15/06–21/06 | EC2, EBS, RDS, mạng và kết nối database | EC2 hoạt động, PostgreSQL riêng tư sẵn sàng |
| **Tuần 4** | 22/06–28/06 | FastAPI, schema, biến môi trường và systemd | Backend chạy và kết nối RDS |
| **Tuần 5** | 29/06–05/07 | Telemetry, latest, history, polling lệnh và ACK | Hoàn tất REST và lưu trạng thái lệnh |
| **Tuần 6** | 06/07–12/07 | Firmware YOLO UNO, cảm biến, thiết bị chấp hành, Wi-Fi và REST | Phần cứng gửi telemetry, xử lý 8 lệnh |
| **Tuần 7** | 13/07–19/07 | Dashboard React, biểu đồ, điều khiển, tích hợp và sửa lỗi | Frontend đọc dữ liệu AWS, tạo lệnh thành công |
| **Tuần 8** | 20/07–31/07 | Kiểm thử đầu cuối, CloudWatch, bảo mật, tài liệu, demo và bàn giao | Hoàn tất bằng chứng và tài liệu song ngữ để nộp |

---

## 7. Cấu hình tài nguyên, chi phí và tối ưu

### 7.1 Cấu hình hiện tại

| Tài nguyên | Cấu hình | Yếu tố chi phí | Hướng tối ưu |
|---|---|---|---|
| EC2 | `t3.micro`, Linux, Public IPv4, Single-AZ | Thời gian chạy và lưu lượng | Dừng khi không dùng; xem CPU/bộ nhớ trước khi đổi cấu hình |
| EBS | Root volume `gp3` 10 GiB | Dung lượng và snapshot | Chỉ giữ dung lượng cần thiết, xóa snapshot không dùng |
| RDS PostgreSQL | `db.t4g.micro`, Single-AZ, không public | Thời gian chạy, storage, backup, lưu lượng | Dừng lúc rảnh nếu được phép; rà soát backup và kết nối |
| RDS storage | General Purpose SSD 20 GiB | Dung lượng cấp phát | Theo dõi tăng trưởng và tránh lưu thừa |
| CloudWatch | Log, metric EC2/RDS, dashboard, 5 alarm | Thu thập/giữ log, custom metric, dashboard, alarm | Đặt retention phù hợp, xóa thành phần không dùng |
| Data transfer | HTTP telemetry và polling | Lưu lượng Internet/dữ liệu | Chọn chu kỳ hợp lý, dùng payload gọn |
| IAM/VPC | Role, VPC, subnet, route, IGW, Security Group | Cấu hình cơ bản chủ yếu không có phí theo giờ; tài nguyên gắn kèm và xử lý dữ liệu có thể có phí | Xóa phụ thuộc không dùng một cách có kiểm soát |

Proposal không đưa ra tổng phí cố định vì chi phí phụ thuộc thời gian chạy, Region, lưu lượng, retention, backup và giá của tài khoản. Cần kiểm tra chi phí thực tế trong AWS Billing and Cost Management.

### 7.2 Hướng tối ưu

- Dừng EC2/RDS khi không cần demo và chính sách cho phép.
- Dùng CloudWatch để đánh giá cấu hình tài nguyên.
- Chỉ giữ log trong thời gian cần cho chấm bài và xử lý sự cố.
- Cân bằng độ phản hồi và số request polling.
- Clean-up theo thứ tự phụ thuộc trong Workshop mục 5.10.

---

## 8. Đánh giá rủi ro

| Rủi ro | Khả năng | Ảnh hưởng | Biện pháp giảm thiểu |
|---|---|---|---|
| Public IPv4 của EC2 đổi sau stop/start | Trung bình | Cao | Cập nhật frontend/firmware và ghi lại endpoint đang dùng |
| Workshop dùng HTTP | Cao | Cao | Không truyền dữ liệu nhạy cảm, giới hạn thời gian mở, chuẩn bị HTTPS trước khi mở rộng |
| Security Group quá rộng | Trung bình | Cao | Giới hạn SSH; giữ RDS `5432` chỉ nhận từ EC2 Security Group |
| Credential bị commit | Thấp | Cao | Dùng secret cục bộ, file mẫu placeholder và kiểm tra Git status |
| EC2 không kết nối được RDS | Trung bình | Cao | Kiểm tra subnet, DB Subnet Group, DNS, SG và TLS bằng `psql` |
| Thiết bị mất Wi-Fi/backend | Trung bình | Trung bình | Bổ sung retry/reconnect và hiển thị trạng thái kết nối |
| Polling tạo nhiều request | Trung bình | Trung bình | Dùng chu kỳ đã thống nhất, xem browser/backend log |
| Lệnh kẹt ở `Pending` | Trung bình | Cao | Kiểm tra polling, tên/ID lệnh, actuator, ACK và database |
| GPIO hoặc nguồn không ổn định | Trung bình | Cao | Dùng pin map đúng, chung GND, nguồn an toàn, test từng phần |
| Bằng chứng giám sát thiếu | Trung bình | Trung bình | Kiểm tra Agent, log group, metric, retention và 5 alarm |
| Tài nguyên tiếp tục phát sinh phí | Trung bình | Cao | Phân công clean-up và làm đúng thứ tự phụ thuộc |

---

## 9. Kết quả kỳ vọng và tiêu chí thành công

### 9.1 Kết quả kỳ vọng

Mô hình cần tạo một luồng có thể truy vết từ cảm biến đến lưu trữ AWS và dashboard, đồng thời có luồng ngược từ lệnh đến thiết bị chấp hành và ACK. Bằng chứng phải cho phép người đánh giá kiểm tra tầng ứng dụng, database, phần cứng và giám sát.

### 9.2 Tiêu chí thành công đo được

| Tiêu chí | Bằng chứng nghiệm thu |
|---|---|
| Backend chạy | `systemctl status aws-iot-backend` hiển thị `active (running)` |
| Health hoạt động | `/api/health` trả `status: ok` |
| EC2 kết nối RDS | `psql` dùng SSL/TLS và `\dt` có các bảng ứng dụng |
| Telemetry được lưu | PostgreSQL có bản ghi của `room_01` |
| Dashboard có dữ liệu | Nhiệt độ, độ ẩm, ánh sáng hiển thị Live AWS |
| History hoạt động | Biểu đồ dùng `/api/devices/room_01/history` |
| Frontend gọi API thành công | Latest, history, command trả HTTP `200` |
| Lệnh được lưu | `commands` có lệnh và trạng thái tương ứng |
| Thiết bị phản ứng | Quạt, đèn, servo rèm hoạt động trong demo |
| Vòng đời ACK hoạt động | Cùng một lệnh chuyển `Pending` → `Executed` |
| CloudWatch Logs hoạt động | Log stream có request log FastAPI |
| Metric hiển thị | Có EC2 CPU/disk, RDS CPU/connections và EC2 memory do Agent thu thập |
| Alarm được cấu hình | CloudWatch hiển thị 5 alarm |
| RDS không public | Internet access disabled hoặc `Publicly accessible: No` |
| PostgreSQL được giới hạn | RDS chỉ nhận TCP `5432` từ EC2 Security Group |
| Secret không lên Git | `.env`, `secrets.h`, `*.pem`, credential thật không được track |

---

## 10. Hạn chế hiện tại và hướng phát triển

### 10.1 Hạn chế hiện tại

- Một EC2 có Public IPv4 và một RDS Single-AZ.
- Frontend chạy cục bộ ngoài AWS; Workshop dùng HTTP và chưa có xác thực.
- Nhận lệnh và cập nhật dashboard dựa trên polling.
- Một phòng mẫu mang mã định danh `room_01`.
- Đề xuất dựa trên luật, không phải machine learning đã huấn luyện.
- Triển khai và kiểm thử thủ công.

### 10.2 Hướng phát triển

Sau khi đánh giá yêu cầu, bảo mật, chi phí và độ phức tạp, dự án có thể bổ sung HTTPS và xác thực, ranh giới mạng chặt chẽ hơn, nhiều phòng và định danh, quy trình sẵn sàng/khôi phục, giao tiếp hướng sự kiện khi quy mô lớn hơn, tự động hóa kiểm thử/triển khai, quản lý secret và backup tốt hơn, thông báo vận hành và giao diện thân thiện với thiết bị di động.

Đây là các phương án tương lai, chưa phải thành phần của mô hình hiện tại.

---

## 11. Phân công thành viên

| Thành viên | Vai trò và trách nhiệm |
|---|---|
| **Phạm Lê Minh Khôi** | Hạ tầng AWS, EC2, EBS, RDS, VPC, Security Group, IAM, CloudWatch, DevOps, phát triển firmware PlatformIO và tích hợp phần cứng YOLO UNO |
| **Thượng Đình Hưng** | Frontend React + Vite, dashboard, tích hợp tổng thể, sửa lỗi và hỗ trợ ghi hình demo |
| **Ngô Minh Thuận** | FastAPI backend, REST endpoint, tích hợp PostgreSQL, xử lý telemetry, lệnh và ACK |
| **Lê Bảo Khánh** | Tài liệu và QA, Proposal, Blog, Worklog, Event Report và nội dung Workshop song ngữ |

---

## 12. Sản phẩm bàn giao và minh chứng

| Sản phẩm | Nội dung | Minh chứng |
|---|---|---|
| GitHub repository | Backend, frontend, hardware, sơ đồ, README song ngữ | [Source Code]({{% relref "8-References/8.1-source-code/_index.vi.md" %}}), nhánh `main` công khai |
| FastAPI backend | Health, telemetry, history, tạo lệnh, polling, ACK | [Workshop Backend]({{% relref "5-Workshop/5.5-Backend-and-Database/_index.vi.md" %}}), systemd và health |
| PostgreSQL database | Bảng ứng dụng, telemetry, trạng thái lệnh | `psql`, `\dt`, query telemetry và lệnh |
| Frontend React + Vite | Telemetry, điều khiển, đề xuất dựa trên luật, biểu đồ | [Workshop Frontend]({{% relref "5-Workshop/5.7-Frontend-Integration/_index.vi.md" %}}), ảnh dashboard |
| Firmware YOLO UNO | Cảm biến, actuator, chế độ, polling, ACK | [Workshop Phần cứng]({{% relref "5-Workshop/5.6-Hardware-Integration/_index.vi.md" %}}), source, build, demo |
| Kiểm thử đầu cuối | Lưu telemetry, vòng đời lệnh, thực thi vật lý | [Workshop Kiểm thử]({{% relref "5-Workshop/5.8-End-to-End-Testing/_index.vi.md" %}}), [Video Demo]({{% relref "8-References/8.2-demo-video/_index.vi.md" %}}) |
| Giám sát CloudWatch | Log, metric, dashboard, 5 alarm | [Workshop CloudWatch]({{% relref "5-Workshop/5.9-CloudWatch-Monitoring/_index.vi.md" %}}), ảnh chụp |
| Workshop song ngữ | Triển khai, tích hợp, kiểm thử, giám sát, bàn giao | [Workshop]({{% relref "5-Workshop/_index.vi.md" %}}) |
| Hướng dẫn clean-up | Tài nguyên, chi phí, bảo mật, thứ tự xóa | [Chi phí, Bảo mật và Clean-up]({{% relref "5-Workshop/5.10-Cost-Security-Cleanup/_index.vi.md" %}}) |
| Bộ tài liệu tham khảo | Source code, demo, tài liệu và liên kết kỹ thuật | [Tài liệu tham khảo]({{% relref "8-References/_index.vi.md" %}}) |

Các liên kết công khai:

- [Source code dự án](https://github.com/toniminhkhoi/aws-iot-dashboard/tree/main)
- [Video demo đầu cuối](https://drive.google.com/file/d/1T97dUY58hbT2ppxvg7ESR12Jg9BA828W/view?usp=sharing)