---
title: "Bản đề xuất"
date: "2026-06-15"
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# AWS IoT Monitoring and Control Dashboard

### Bản tải xuống: <a href="/files/2-Proposal/IoT_Dashboard_Proposal_vi.pdf" download>IoT Dashboard Proposal (PDF)</a>



---

## 1. Tóm tắt dự án

**AWS IoT Monitoring and Control Dashboard** giải quyết nhu cầu theo dõi điều kiện môi trường và điều khiển thiết bị trong phòng từ một giao diện tập trung. Hệ thống kết nối phần cứng YOLO UNO với backend triển khai trên AWS, cho phép người dùng xem nhiệt độ, độ ẩm và ánh sáng, gửi lệnh điều khiển từ xa, đồng thời xác minh thiết bị vật lý đã thực thi thông qua cơ chế xác nhận lệnh.

Trong mô hình Workshop hiện tại:

- YOLO UNO thu thập telemetry môi trường và điều khiển quạt, đèn, servo rèm;
- CloudFront và AWS WAF phân phối bản build React + Vite từ Amazon S3 private;
- request browser `/api/*` đi qua CloudFront tới Application Load Balancer;
- Auto Scaling Group duy trì hai EC2 FastAPI chạy `aws-iot-backend.service`;
- Amazon RDS for PostgreSQL Multi-AZ lưu telemetry và trạng thái lệnh;
- YOLO UNO dùng trực tiếp DNS ALB cho telemetry, polling và ACK;
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

- Triển khai FastAPI, Uvicorn, SQLAlchemy và Pydantic trên hai EC2 do ASG quản lý phía sau ALB.
- Quản lý backend bằng `aws-iot-backend.service`.
- Lưu telemetry và lệnh trong database `iot_dashboard` trên RDS PostgreSQL.
- Cấu hình VPC, định tuyến và Security Group phù hợp.
- Giữ RDS không public, chỉ nhận PostgreSQL từ Security Group của EC2.
- Dùng IAM Role/Instance Profile cho quyền của CloudWatch Agent.
- Phân phối frontend S3 private qua CloudFront OAC và giám sát bằng AWS WAF managed rules.
- Dùng RDS PostgreSQL Multi-AZ với automated backup và manual snapshot.

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

Mô hình dùng `room_01` làm mã định danh logic cho phòng mẫu. Phạm vi gồm ba loại dữ liệu môi trường, ba thiết bị chấp hành, HTTP REST, polling lệnh, ACK, frontend CloudFront/WAF/S3 private, ALB với ASG gồm hai FastAPI instance, RDS PostgreSQL Multi-AZ và CloudWatch.

### 3.6 Ngoài phạm vi

Phiên bản hiện tại chưa gồm triển khai nhiều phòng production, xác thực người dùng/thiết bị, ALB origin HTTPS với custom domain, scaling policy theo tải ngoài dung lượng ASG đã cấu hình, diễn tập failover đã kiểm thử, ứng dụng di động, pipeline triển khai tự động hoặc mô hình machine learning đã huấn luyện. Các khả năng này cần đánh giá riêng về yêu cầu, chi phí, bảo mật và độ phức tạp.

---

## 4. Kiến trúc giải pháp

![Kiến trúc AWS IoT Monitoring and Control Dashboard](/images/2-Proposal/IoT_Dashboard_Architecture.png)

*Kiến trúc hiện tại gồm CloudFront/WAF/S3 private, backend FastAPI qua ALB/ASG, RDS PostgreSQL Multi-AZ, YOLO UNO và CloudWatch.*

### 4.1 Vị trí tài nguyên

| Vị trí | Thành phần | Trách nhiệm |
|---|---|---|
| Ngoài AWS | Người dùng dashboard, YOLO UNO ESP32-S3 | Tương tác; telemetry thiết bị, thực thi lệnh và ACK |
| AWS edge, ngoài VPC | CloudFront, AWS WAF, S3 private origin | Viewer HTTPS, phân phối frontend, route `/api/*` và WAF monitoring |
| AWS Cloud, ngoài VPC | IAM Role/Instance Profile của EC2, CloudWatch | Quyền, log, metric, dashboard và alarm |
| Amazon VPC | Internet Gateway, hai application subnet, các DB subnet private | Ranh giới mạng và định tuyến |
| Application subnet 1a/1c | ALB Internet-facing, target group, EC2 trong ASG | Điểm vào HTTP và khả năng sẵn sàng của FastAPI |
| DB subnet private 1c/1b | Amazon RDS for PostgreSQL Multi-AZ | Lưu telemetry/lệnh riêng tư và standby failover |
| Gắn với các EC2 của ASG | Root volume EBS `gp3` 10 GiB đã mã hóa | Hệ điều hành, ứng dụng và file runtime |

CloudWatch Agent chạy trên từng backend instance để gửi log và guest metric. ALB, ASG, EC2 và RDS xuất managed metric. RDS standby dùng cho failover, không phải read replica của ứng dụng.

### 4.2 Luồng telemetry

1. YOLO UNO đọc DHT20 và cảm biến ánh sáng analog.
2. Firmware gửi `POST /api/telemetry` trực tiếp tới DNS ALB.
3. FastAPI kiểm tra rồi lưu bản ghi vào RDS PostgreSQL.
4. Frontend trên CloudFront gọi endpoint tương đối `/api/*` latest/history; CloudFront forward tới ALB.
5. Dashboard hiển thị dữ liệu hiện tại và lịch sử.

### 4.3 Luồng lệnh và xác nhận

1. Người vận hành tạo lệnh từ frontend trên CloudFront qua `/api/*`.
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
- S3 frontend bật Block Public Access và được CloudFront đọc qua OAC.
- CloudFront cung cấp viewer HTTPS; ba WAF managed rule group hiện chạy ở Count/Monitor.
- ALB Security Group nhận HTTP:80; backend port 8000 chỉ nhận từ ALB Security Group.
- Cổng TCP `5432` của RDS chỉ nhận từ Security Group của EC2, không nhận public CIDR.
- EC2 dùng IAM Role/Instance Profile cho CloudWatch thay vì AWS credential tĩnh.
- Giá trị runtime được cung cấp qua biến môi trường hoặc file secret cục bộ.
- Không commit `backend/.env`, `hardware/include/secrets.h`, `*.pem`, mật khẩu hoặc credential.

---

## 5. Kế hoạch triển khai kỹ thuật

### Giai đoạn 1 — Yêu cầu và nền tảng AWS

Thống nhất yêu cầu chức năng và tiêu chí nghiệm thu, phân công trách nhiệm và quy ước `room_01` là mã định danh logic. Sau đó, thiết kế luồng VPC/Security Group, triển khai backend/database ban đầu và kiểm tra kết nối riêng tư.

### Giai đoạn 2 — Backend và cơ sở dữ liệu

Thiết kế schema PostgreSQL; xây dựng health, telemetry, latest, history, tạo lệnh, polling và ACK; cấu hình biến môi trường và systemd.

### Giai đoạn 3 — Phần cứng và firmware

Kết nối DHT20, cảm biến ánh sáng, quạt, đèn/relay, servo và LCD; phát triển firmware PlatformIO cho cảm biến, Wi-Fi, telemetry, polling, điều khiển, chế độ và ACK; xác minh đủ 8 lệnh.

### Giai đoạn 4 — Frontend và tích hợp

Xây dựng dashboard React + Vite + TypeScript, biểu đồ, điều khiển và đề xuất dựa trên luật; deploy lên S3 private qua CloudFront/WAF; thêm route `/api/*` tới ALB; xử lý lỗi API, CORS, trạng thái lệnh và phần cứng.

### Giai đoạn 5 — Giám sát, kiểm thử và bàn giao

Tạo AMI/Launch Template backend, ALB/target group/ASG, RDS Multi-AZ và backup; cấu hình CloudWatch Agent, log, metric và 8 alarm; sau đó kiểm thử đầu cuối, rà bảo mật, hoàn thiện Workshop song ngữ, bằng chứng, demo, clean-up và bàn giao.

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
| **Tuần 8** | 20/07–31/07 | CloudFront/WAF/S3, ALB/ASG, RDS Multi-AZ, kiểm thử đầu cuối, CloudWatch, bảo mật, tài liệu, demo và bàn giao | Xác minh kiến trúc hiện tại và hoàn tất tài liệu song ngữ để nộp |

---

## 7. Cấu hình tài nguyên, chi phí và tối ưu

### 7.1 Cấu hình hiện tại

| Tài nguyên | Cấu hình | Yếu tố chi phí | Hướng tối ưu |
|---|---|---|---|
| S3 / CloudFront / WAF | S3 private origin, CloudFront distribution, ba WAF group Count mode | Storage, request, transfer, plan/WAF usage | Chỉ giữ artifact build; rà giới hạn plan và WAF mode |
| Application Load Balancer | Internet-facing, HTTP:80, target group hai AZ | Thời gian ALB và capacity unit | Xóa sau đánh giá nếu không còn cần |
| EC2 / Auto Scaling | Hai `t3.micro` Linux, ASG `2/2/4` | Giờ instance và truyền dữ liệu | Rà desired capacity/metric; xóa ASG sau bàn giao nếu được duyệt |
| EBS | Root volume `gp3` 10 GiB mã hóa trên mỗi instance ASG | Dung lượng và snapshot | Chỉ giữ dung lượng cần, xóa volume/snapshot không dùng |
| RDS PostgreSQL | `db.t4g.micro`, Multi-AZ, không public | Giờ Multi-AZ, storage, backup, transfer | Rà retention/kết nối; xóa sau khi lưu bằng chứng được duyệt |
| RDS storage | General Purpose SSD 20 GiB | Dung lượng cấp phát | Theo dõi tăng trưởng và tránh lưu thừa |
| CloudWatch | Log, metric ALB/ASG/EC2/RDS, dashboard, 8 alarm | Thu thập/giữ log, custom metric, dashboard, alarm | Đặt retention phù hợp, xóa thành phần không dùng |
| Data transfer | CloudFront/API/telemetry thiết bị và polling | Lưu lượng Internet, edge và liên AZ | Chọn chu kỳ hợp lý, payload gọn và chỉ cache nội dung tĩnh |
| IAM/VPC | Role, VPC, subnet, route, IGW, Security Group | Cấu hình cơ bản chủ yếu không có phí theo giờ; tài nguyên gắn kèm và xử lý dữ liệu có thể có phí | Xóa phụ thuộc không dùng một cách có kiểm soát |

Proposal không đưa ra tổng phí cố định vì chi phí phụ thuộc thời gian chạy, Region, lưu lượng, retention, backup và giá của tài khoản. Cần kiểm tra chi phí thực tế trong AWS Billing and Cost Management.

### 7.2 Hướng tối ưu

- Dùng clean-up theo phụ thuộc thay vì dừng một instance ASG vì ASG sẽ tạo lại khi desired capacity vẫn là 2.
- Xóa RDS khi kết thúc môi trường; dừng chỉ là tạm thời và phụ thuộc hành vi dịch vụ.
- Dùng CloudWatch để đánh giá cấu hình tài nguyên.
- Chỉ giữ log trong thời gian cần cho chấm bài và xử lý sự cố.
- Cân bằng độ phản hồi và số request polling.
- Clean-up theo thứ tự phụ thuộc trong Workshop mục 5.10.

---

## 8. Đánh giá rủi ro

| Rủi ro | Khả năng | Ảnh hưởng | Biện pháp giảm thiểu |
|---|---|---|---|
| ALB origin/route thiết bị dùng HTTP | Cao | Cao | Không truyền dữ liệu nhạy cảm; bổ sung ACM/custom domain và TLS trước khi mở rộng |
| WAF Count mode không chặn | Trung bình | Cao | Rà sampled request/false positive rồi bật Block theo giai đoạn đã kiểm thử |
| Security Group quá rộng | Trung bình | Cao | Giới hạn SSH; backend 8000 chỉ nhận từ ALB SG và RDS 5432 chỉ nhận từ backend SG |
| Credential bị commit | Thấp | Cao | Dùng secret cục bộ, file mẫu placeholder và kiểm tra Git status |
| EC2 không kết nối được RDS | Trung bình | Cao | Kiểm tra subnet, DB Subnet Group, DNS, SG và TLS bằng `psql` |
| Thiết bị mất Wi-Fi/backend | Trung bình | Trung bình | Bổ sung retry/reconnect và hiển thị trạng thái kết nối |
| Polling tạo nhiều request | Trung bình | Trung bình | Dùng chu kỳ đã thống nhất, xem browser/backend log |
| Lệnh kẹt ở `Pending` | Trung bình | Cao | Kiểm tra polling, tên/ID lệnh, actuator, ACK và database |
| GPIO hoặc nguồn không ổn định | Trung bình | Cao | Dùng pin map đúng, chung GND, nguồn an toàn, test từng phần |
| Deployment ASG không đồng nhất | Trung bình | Cao | Phát hành bằng AMI/Launch Template có version và kiểm tra instance refresh/target health |
| Failover Multi-AZ chưa kiểm thử | Trung bình | Cao | Lập diễn tập failover có kiểm soát, kiểm tra reconnect endpoint và toàn vẹn dữ liệu |
| Bằng chứng giám sát thiếu | Trung bình | Trung bình | Kiểm tra Agent, log stream, metric, retention và 8 alarm |
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
| Frontend CloudFront hoạt động | Bản build React trong S3 private tải qua CloudFront viewer HTTPS |
| Route browser API hoạt động | Request CloudFront `/api/*` tới ALB origin và trả HTTP `200` |
| ALB/ASG Healthy | Hai target ở hai Availability Zone Healthy và ASG desired capacity bằng 2 |
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
| Alarm được cấu hình | CloudWatch hiển thị 8 alarm |
| RDS Multi-AZ được bật | Bằng chứng CLI/console xác định primary và standby Availability Zone |
| Có kiểm soát backup | Automated retention 7 ngày và manual snapshot ở trạng thái Available |
| RDS không public | Internet access disabled hoặc `Publicly accessible: No` |
| PostgreSQL được giới hạn | RDS chỉ nhận TCP `5432` từ EC2 Security Group |
| Secret không lên Git | `.env`, `secrets.h`, `*.pem`, credential thật không được track |

---

## 10. Hạn chế hiện tại và hướng phát triển

### 10.1 Hạn chế hiện tại

- CloudFront cung cấp viewer HTTPS, nhưng ALB origin và route trực tiếp của thiết bị dùng HTTP; chưa có custom domain được ghi nhận.
- API chưa áp dụng xác thực mạnh cho người dùng/thiết bị hoặc rate limiting.
- WAF managed rules ở Count/Monitor và chưa được kiểm thử ở Block mode.
- Nhận lệnh và cập nhật dashboard dựa trên polling.
- Một phòng mẫu mang mã định danh `room_01`.
- Đề xuất dựa trên luật, không phải machine learning đã huấn luyện.
- Triển khai/kiểm thử phần lớn còn thủ công; chưa có diễn tập failover ALB/ASG hoặc RDS có kiểm soát.

### 10.2 Hướng phát triển

Sau khi đánh giá yêu cầu, bảo mật, chi phí và độ phức tạp, dự án có thể bổ sung custom domain/ALB HTTPS, xác thực, WAF Block đã kiểm thử, nhiều phòng/định danh, diễn tập failover, giao tiếp hướng sự kiện khi mở rộng, tự động hóa kiểm thử/triển khai, managed secrets, thông báo vận hành và giao diện mobile.

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
| AWS edge và tính sẵn sàng | CloudFront/WAF/S3 private, ALB/target group/ASG, RDS Multi-AZ và backup | [Workshop AWS]({{% relref "5-Workshop/5.4-AWS-Infrastructure-Setup/_index.vi.md" %}}), ảnh chụp |
| Giám sát CloudWatch | Log, metric ALB/ASG/EC2/RDS, dashboard, 8 alarm | [Workshop CloudWatch]({{% relref "5-Workshop/5.9-CloudWatch-Monitoring/_index.vi.md" %}}), ảnh chụp |
| Workshop song ngữ | Triển khai, tích hợp, kiểm thử, giám sát, bàn giao | [Workshop]({{% relref "5-Workshop/_index.vi.md" %}}) |
| Hướng dẫn clean-up | Tài nguyên, chi phí, bảo mật, thứ tự xóa | [Chi phí, Bảo mật và Clean-up]({{% relref "5-Workshop/5.10-Cost-Security-Cleanup/_index.vi.md" %}}) |
| Bộ tài liệu tham khảo | Source code, demo, tài liệu và liên kết kỹ thuật | [Tài liệu tham khảo]({{% relref "8-References/_index.vi.md" %}}) |

Các liên kết công khai:

- [Source code dự án](https://github.com/toniminhkhoi/aws-iot-dashboard/tree/main)
- [Video demo đầu cuối](https://drive.google.com/file/d/1T97dUY58hbT2ppxvg7ESR12Jg9BA828W/view?usp=sharing)
