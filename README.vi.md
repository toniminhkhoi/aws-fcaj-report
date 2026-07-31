# AWS IoT Monitoring and Control Dashboard 🚀

🌐 **Language / Ngôn ngữ:** [English](README.md) | [Tiếng Việt](README.vi.md)

Hệ thống giám sát môi trường và điều khiển thiết bị IoT từ xa, sử dụng FastAPI, React, PostgreSQL, YOLO UNO và các dịch vụ AWS.

> Phạm vi hiện tại là một mô hình thử nghiệm cho phòng `room_01`. Dự án chưa phải hệ thống Building Management System (BMS) đa chi nhánh hoặc nền tảng vận hành ở quy mô doanh nghiệp.

---

## 📋 Tổng quan dự án

YOLO UNO thu thập nhiệt độ, độ ẩm và giá trị ánh sáng rồi gửi telemetry qua HTTP trực tiếp đến Application Load Balancer (ALB). ALB chuyển tiếp yêu cầu đến Target Group gồm hai backend FastAPI trong Auto Scaling Group (ASG). Backend lưu telemetry và trạng thái lệnh trong Amazon RDS for PostgreSQL Multi-AZ.

Dashboard React + Vite được build và lưu trong bucket Amazon S3 riêng tư, phân phối qua Amazon CloudFront bằng Origin Access Control (OAC), đồng thời được bảo vệ bởi AWS WAF ở chế độ Count/Monitor. Yêu cầu trình duyệt đến `/api/*` được CloudFront chuyển tiếp sang ALB. Thiết bị thăm dò lệnh đang chờ, thực thi lệnh một lần và gửi ACK để backend chuyển trạng thái từ `Pending` sang `Executed`. Amazon CloudWatch thu thập log backend, theo dõi metric ALB, ASG, EC2, RDS và đánh giá tám cảnh báo đã cấu hình.

- **Người biên soạn báo cáo:** Phạm Lê Minh Khôi
- **Đơn vị:** Trường Đại học Bách Khoa TP.HCM (HCMUT) – Khoa Khoa học và Kỹ thuật Máy tính
- **Workshop:** [Xem báo cáo trực tuyến](https://toniminhkhoi.github.io/aws-fcaj-report/vi)

---

## 🏛️ Kiến trúc hệ thống

```text
Trình duyệt ──HTTPS──> CloudFront + AWS WAF
                         ├── Default (*) ──> S3 riêng tư (React + Vite)
                         └── /api/* ──────> ALB HTTP:80
                                                │
YOLO UNO / Simulator ──HTTP─────────────────────┤
                                                ▼
                                     Target Group HTTP:8000
                                                │
                                     ASG: 2 EC2 instances
                                     EBS · IAM Role · systemd
                                                │
                                                ▼
                                 RDS PostgreSQL Multi-AZ
                                 Primary 1c · Standby 1b

CloudWatch giám sát ALB, ASG, EC2, RDS, log và tám cảnh báo.
```

### Thành phần chính

1. **Amazon CloudFront, AWS WAF và S3 riêng tư:** Phân phối dashboard React + Vite qua HTTPS. Behavior mặc định đọc nội dung S3 riêng tư bằng OAC; behavior `/api/*` chuyển tiếp yêu cầu động đến ALB. Ba nhóm AWS Managed Rules hiện chạy ở chế độ Count/Monitor.
2. **Application Load Balancer, Target Group và Auto Scaling Group:** Phân phối yêu cầu HTTP đến hai target FastAPI khỏe mạnh tại `ap-southeast-1a` và `ap-southeast-1c`. ASG có min/desired/max là `2/2/4`.
3. **Amazon EC2 và EBS mã hóa:** Chạy FastAPI/Uvicorn ở cổng `8000` dưới dạng dịch vụ `systemd`. Launch Template dùng AMI riêng tư và ổ gốc gp3 10 GiB được mã hóa bằng khóa KMS do AWS quản lý `aws/ebs`.
4. **Amazon RDS for PostgreSQL Multi-AZ:** Lưu thiết bị, lịch sử telemetry và vòng đời lệnh. Primary nằm tại `ap-southeast-1c`; standby đồng bộ nằm tại `ap-southeast-1b` và không phải read replica.
5. **YOLO UNO / Python Simulator:** Gửi telemetry, nhận lệnh đang chờ và phản hồi ACK trực tiếp qua DNS của ALB bằng HTTP. Simulator hỗ trợ kiểm thử khi chưa kết nối phần cứng.
6. **Amazon CloudWatch:** Thu thập log backend, theo dõi metric ALB, ASG, EC2, RDS và đánh giá tám cảnh báo. Alarm actions/thông báo SNS chưa được cấu hình.
7. **Amazon VPC, Security Group và IAM Role:** Giới hạn lưu lượng ALB đến backend ở cổng `8000`, backend đến RDS ở cổng `5432`, SSH quản trị và quyền CloudWatch Agent.

Các dịch vụ đang dùng gồm **Amazon S3, Amazon CloudFront, AWS WAF, Elastic Load Balancing, EC2 Auto Scaling, Amazon EC2, Amazon EBS, Amazon RDS for PostgreSQL Multi-AZ, Amazon VPC, Security Group, AWS IAM, Amazon CloudWatch và CloudWatch Alarms**.

AWS IoT Core, Lambda, API Gateway, DynamoDB, Amazon SQS, Secrets Manager và Amazon SNS chưa được triển khai trong phiên bản hiện tại.

---

## 🔄 Luồng hoạt động chính

- **Telemetry từ trình duyệt:** Trình duyệt → HTTPS CloudFront/WAF → behavior `/api/*` → ALB → target FastAPI khỏe mạnh → RDS PostgreSQL → phản hồi theo cùng đường đi.
- **Telemetry từ thiết bị:** YOLO UNO → HTTP đến DNS của ALB → Target Group → FastAPI → RDS PostgreSQL.
- **Lệnh điều khiển:** Người vận hành thao tác trên dashboard → FastAPI tạo lệnh ở trạng thái `Pending` → thiết bị thăm dò qua ALB và thực thi → thiết bị gửi ACK → backend cập nhật trạng thái `Executed`.
- **Giám sát:** CloudWatch Agent gửi log backend và metric hệ điều hành EC2; CloudWatch theo dõi thêm ALB, ASG, EC2 và RDS qua dashboard vận hành và tám cảnh báo.

---

## 👥 Thành viên và trách nhiệm

| Thành viên | Trách nhiệm |
| :--- | :--- |
| **Phạm Lê Minh Khôi** | Kiến trúc AWS; VPC, Security Group, IAM Role, EC2, RDS và CloudWatch; DevOps; phần cứng YOLO UNO; cảm biến, thiết bị chấp hành, telemetry, cơ chế thăm dò lệnh và ACK |
| **Ngô Minh Thuận** | Backend FastAPI; API endpoint, lược đồ Pydantic, mô hình SQLAlchemy, tích hợp PostgreSQL, xử lý telemetry, vòng đời lệnh và ACK |
| **Thượng Đình Hưng** | Frontend React + Vite; giao diện dashboard, trực quan hóa telemetry, chức năng điều khiển, tích hợp tổng thể, xử lý lỗi và quay/dựng video minh họa |
| **Lê Bảo Khánh** | Proposal, blog, nhật ký công việc hằng tuần, báo cáo sự kiện, Workshop song ngữ, điều hướng, ảnh minh họa và bảo đảm chất lượng tài liệu |

Chi tiết đóng góp và phần nhìn lại riêng của từng thành viên được trình bày trong [mục 5.11](content/5-Workshop/5.11-Results-Challenges-Future/_index.vi.md) và [mục 5.12](content/5-Workshop/5.12-Project-Handover/_index.vi.md).

---

## 🧰 Công nghệ sử dụng

- **Backend:** Python, FastAPI, Uvicorn, SQLAlchemy và Pydantic.
- **Frontend:** React, Vite, TypeScript và Tailwind CSS.
- **Cơ sở dữ liệu:** PostgreSQL trên Amazon RDS.
- **Phần cứng:** YOLO UNO/ESP32-S3, PlatformIO, DHT20, cảm biến ánh sáng analog, LCD1602, quạt, đèn/relay và servo.
- **AWS:** S3, CloudFront, WAF, ALB, Target Group, EC2 Auto Scaling, EC2, EBS mã hóa, RDS PostgreSQL Multi-AZ, VPC, Security Group, IAM Role, CloudWatch và CloudWatch Alarms.
- **Báo cáo:** Hugo và `hugo-theme-learn`.

---

## 📚 Nội dung báo cáo

- `content/1-Worklog/`: Nhật ký công việc.
- `content/2-Proposal/`: Đề xuất dự án.
- `content/3-BlogsTranslated/`: Bài viết đã dịch.
- `content/4-EventParticipated/`: Sự kiện đã tham gia.
- `content/5-Workshop/`: Workshop AWS IoT Dashboard bằng tiếng Anh và tiếng Việt.
- `content/6-Self-evaluation/`: Tự đánh giá.
- `content/7-Feedback/`: Phản hồi.

Mã nguồn backend, frontend và firmware được quản lý trong kho ứng dụng `aws-iot-dashboard`; kho hiện tại chứa báo cáo thực tập và Workshop.

---

## ▶️ Chạy website bằng Hugo

```powershell
hugo server
```

Sau đó mở `http://localhost:1313/`. Bộ chọn ngôn ngữ cho phép chuyển giữa **English** và **Tiếng Việt**.

Để tạo bản dựng tĩnh:

```powershell
hugo --minify
```

---

## 🔐 Lưu ý bảo mật

- Không đưa `.env`, `.pem`, khóa riêng, mật khẩu hoặc `hardware/include/secrets.h` lên Git.
- Giữ bucket frontend S3 ở chế độ riêng tư và chỉ cho phép CloudFront OAC đọc đối tượng.
- Dùng HTTPS cho viewer. Đường CloudFront đến ALB và YOLO UNO đến ALB hiện vẫn dùng HTTP và được ghi nhận là giới hạn.
- Chỉ cho phép Security Group của ALB truy cập cổng backend `8000`.
- Chỉ cho phép RDS nhận kết nối PostgreSQL từ Security Group của EC2.
- Giới hạn SSH theo địa chỉ IP quản trị.
- Không ghi cố định AWS access key trong mã nguồn.
- Dùng IAM Role cho quyền của CloudWatch Agent.
- Bật mã hóa EBS cho mọi instance của ASG.
- Xem WAF Count/Monitor là chế độ quan sát; yêu cầu chưa bị chặn cho đến khi chủ động chuyển rule sang Block.

---

## 📄 Bản quyền

Copyright © 2026 Phạm Lê Minh Khôi – HCMUT. All rights reserved.
