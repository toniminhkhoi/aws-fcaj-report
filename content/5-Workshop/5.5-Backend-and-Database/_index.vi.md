---
title: "Triển khai backend và tích hợp cơ sở dữ liệu"
date: "2026-07-28"
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

## Tổng quan và mục tiêu

Triển khai FastAPI trong môi trường ảo Python trên Amazon EC2, quản lý backend bằng `aws-iot-backend.service` và kết nối tới database `iot_dashboard` trên Amazon RDS for PostgreSQL. Quy trình sử dụng tài khoản `ec2-user`, thư mục `/home/ec2-user/aws-iot-dashboard/backend`, môi trường ảo `venv` và điểm vào Uvicorn `main:app`. Hoàn tất các bước này trên instance nguồn trước khi tạo AMI backend, hoặc phát hành đồng nhất bằng AMI mới và ASG instance refresh.

## Bước 1 - Triển khai FastAPI backend

Kết nối từ Windows PowerShell, cài công cụ, sao chép repository và tạo môi trường ảo:

```powershell
ssh -i "$env:USERPROFILE\.ssh\<KEY_FILE>.pem" ec2-user@<EC2_PUBLIC_IP>
```

```bash
sudo dnf update -y
sudo dnf install -y git python3 python3-pip postgresql15 curl
git clone <REPOSITORY_URL> ~/aws-iot-dashboard
cd ~/aws-iot-dashboard/backend
python3 -m venv venv
source venv/bin/activate
python -m pip install --upgrade pip
pip install -r requirements.txt
```

Tạo file `.env` đã được loại khỏi Git. Không commit thông tin xác thực:

```dotenv
DATABASE_URL=postgresql://<DB_USER>:<DB_PASSWORD>@<RDS_ENDPOINT>:5432/iot_dashboard?sslmode=require
```

Có thể chạy thủ công trong lần kiểm tra triển khai đầu tiên:

```bash
source venv/bin/activate
uvicorn main:app --host 0.0.0.0 --port 8000
```

## Bước 2 - Cấu hình dịch vụ systemd

Tạo `/etc/systemd/system/aws-iot-backend.service`:

```ini
[Unit]
Description=AWS IoT FastAPI backend
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=ec2-user
WorkingDirectory=/home/ec2-user/aws-iot-dashboard/backend
EnvironmentFile=/home/ec2-user/aws-iot-dashboard/backend/.env
ExecStart=/home/ec2-user/aws-iot-dashboard/backend/venv/bin/uvicorn main:app --host 0.0.0.0 --port 8000
Restart=on-failure
RestartSec=5
StandardOutput=append:/var/log/aws-iot-backend/backend.log
StandardError=append:/var/log/aws-iot-backend/backend-error.log

[Install]
WantedBy=multi-user.target
```

Tạo thư mục log, bật và khởi động dịch vụ:

```bash
sudo install -d -o ec2-user -g ec2-user /var/log/aws-iot-backend
sudo systemctl daemon-reload
sudo systemctl enable aws-iot-backend
sudo systemctl restart aws-iot-backend
```

Sau khi được `enable`, systemd sẽ khởi động backend cùng EC2; không cần chạy Uvicorn thủ công sau mỗi lần máy khởi động lại.

## Bước 3 - Xác minh dịch vụ backend

Chạy:

```bash
sudo systemctl status aws-iot-backend --no-pager -l
curl http://127.0.0.1:8000/api/health
```

<p align="center">
  <img src="/images/5-Workshop/5.5-backend-database/backend-systemd-health-check.png"
       alt="Dịch vụ systemd của FastAPI backend và kết quả health check"
       width="100%" />
</p>

*Hình 8. Dịch vụ `aws-iot-backend.service` đang ở trạng thái `active (running)` và endpoint `/api/health` trả về trạng thái `ok`.*

Hình 8 cho thấy unit đã được systemd nạp, dịch vụ ở trạng thái `active (running)` và Uvicorn là tiến trình chính. Endpoint kiểm tra cũng trả về JSON hợp lệ với `"status":"ok"`. Các dữ kiện này chứng minh backend đã được triển khai và có thể nhận HTTP request cục bộ; chúng không chứng minh High Availability hoặc hệ thống không thể gặp lỗi.

Sau khi các instance của ASG được đăng ký, kiểm tra cùng endpoint qua ALB:

```powershell
curl.exe -sS -i "http://<ALB_DNS_NAME>/api/health"
```

![Kiểm tra health qua Application Load Balancer](/images/5-Workshop/5.5-backend-database/alb-health-check.png)
*Hình 8a. Endpoint ALB trả HTTP 200 cho `/api/health`; bằng chứng target group ở mục 5.4 xác nhận cả hai backend đã đăng ký đều Healthy.*

## Bước 4 - Kết nối EC2 với Amazon RDS

Từ EC2, kết nối và yêu cầu SSL/TLS:

```bash
psql "host=<RDS_ENDPOINT> port=5432 dbname=iot_dashboard user=<DB_USER> sslmode=require"
```

Dùng endpoint của RDS, không dùng hostname gắn với một Availability Zone. Khi Multi-AZ failover, AWS ánh xạ lại endpoint này sang standby; standby không phải read replica cho ứng dụng.

Trong `psql`, kiểm tra database và thông tin kết nối mà không hiển thị mật khẩu:

```sql
SELECT current_database(), current_user;
\conninfo
\dt
```

Nếu schema chưa được khởi tạo, chạy lệnh do mã nguồn định nghĩa trong môi trường ảo rồi kiểm tra lại:

```bash
cd ~/aws-iot-dashboard/backend
source venv/bin/activate
python -m app.database.init_db
```

## Bước 5 - Xác minh các bảng và command trong PostgreSQL

Database `iot_dashboard` trong bằng chứng có bốn bảng `commands`, `devices`, `sensor_readings` và `telemetry_logs`. Truy vấn các command gần nhất:

```sql
SELECT id, device_id, command, state, timestamp
FROM commands
ORDER BY id DESC
LIMIT 6;
```

<p align="center">
  <img src="/images/5-Workshop/5.5-backend-database/postgresql-tables-and-commands.png"
       alt="Các bảng PostgreSQL và command IoT ở trạng thái Executed"
       width="100%" />
</p>

*Hình 9. Kết nối từ EC2 đến Amazon RDS PostgreSQL, danh sách các bảng của database và các command gần nhất ở trạng thái `Executed`.*

Ảnh xác nhận một phiên PostgreSQL dùng SSL/TLS từ EC2 tới database `iot_dashboard`. Bốn bảng ứng dụng và các command gần nhất có `device_id` là `room_01` được hiển thị. Các ví dụ gồm `CURTAIN_CLOSE`, `CURTAIN_OPEN`, `MODE_AUTO` và `LIGHT_OFF`, đều ở trạng thái `Executed`. Ảnh không hiển thị mật khẩu database.

![Phản hồi telemetry API và bản ghi PostgreSQL tương ứng](/images/5-Workshop/5.5-backend-database/telemetry-api-database.png)
*Hình 9a. Telemetry POST và latest API response đối chiếu được với cùng bản ghi `telemetry_logs` trong PostgreSQL.*

## Bước 6 - Kết quả mong đợi

- `aws-iot-backend.service` đã được bật và ở trạng thái `active (running)`.
- Uvicorn là tiến trình chính của backend.
- `GET /api/health` trả JSON có trạng thái `ok`.
- Health request qua ALB trả HTTP 200 và cả hai target của ASG duy trì Healthy.
- EC2 kết nối được Amazon RDS for PostgreSQL bằng SSL/TLS.
- `commands`, `devices`, `sensor_readings` và `telemetry_logs` xuất hiện trong `psql`.
- Truy vấn trả về các command có `device_id` là `room_01`.
- Không có mật khẩu, access key hoặc thông tin xác thực khác trong lệnh và ảnh chụp.

## Xử lý sự cố

| Hiện tượng | Chẩn đoán và khắc phục |
| :--- | :--- |
| Dịch vụ backend không chạy | Xem `systemctl status` và `journalctl -u aws-iot-backend`; kiểm tra tài khoản, đường dẫn, `.env` và module Uvicorn |
| Cổng 8000 đã được dùng | Chạy `sudo ss -ltnp \| grep :8000` và dừng tiến trình ngoài dự kiến |
| Health check bị từ chối | Xác nhận dịch vụ đang chạy và Uvicorn lắng nghe trên cổng 8000 |
| Kết nối RDS hết thời gian chờ | Kiểm tra endpoint, Region, đường mạng và nguồn Security Group trên cổng 5432 |
| PostgreSQL xác thực thất bại | Kiểm tra tên database, user, cách mã hóa mật khẩu và `.env` mà systemd nạp |
| Kết nối SSL thất bại | Kiểm tra hostname endpoint và `sslmode`; dùng đúng RDS CA bundle khi bật xác minh chứng chỉ |
| Thiếu bảng | Chạy quy trình khởi tạo do mã nguồn định nghĩa và kiểm tra đúng database `iot_dashboard` |

Tiếp theo: [tích hợp phần cứng YOLO UNO](../5.6-Hardware-Integration/).
