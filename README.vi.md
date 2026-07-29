# AWS IoT Monitoring and Control Dashboard 🚀

🌐 **Language / Ngôn ngữ:** [English](README.md) | [Tiếng Việt](README.vi.md)

Hệ thống giám sát môi trường và điều khiển thiết bị IoT từ xa, sử dụng FastAPI, React, PostgreSQL, YOLO UNO và các dịch vụ AWS.

> Phạm vi hiện tại là một mô hình thử nghiệm cho phòng `room_01`. Dự án chưa phải hệ thống Building Management System (BMS) đa chi nhánh hoặc nền tảng vận hành ở quy mô doanh nghiệp.

---

## 📋 Tổng quan dự án

YOLO UNO thu thập nhiệt độ, độ ẩm và giá trị ánh sáng rồi gửi telemetry qua HTTP đến backend FastAPI trên Amazon EC2. Backend lưu dữ liệu và trạng thái lệnh trong Amazon RDS for PostgreSQL. Dashboard React + Vite hiển thị dữ liệu mới nhất, lịch sử và cho phép người vận hành điều khiển quạt, đèn và rèm.

Thiết bị thăm dò lệnh đang chờ, thực thi lệnh một lần và gửi ACK để backend chuyển trạng thái từ `Pending` sang `Executed`. Amazon CloudWatch thu thập log backend, theo dõi các metric của EC2/RDS và đánh giá các cảnh báo đã cấu hình.

- **Người biên soạn báo cáo:** Phạm Lê Minh Khôi
- **Đơn vị:** Trường Đại học Bách Khoa TP.HCM (HCMUT) – Khoa Khoa học và Kỹ thuật Máy tính
- **Workshop:** [Xem báo cáo trực tuyến](https://toniminhkhoi.github.io/aws-fcaj-report/vi)

---

## 🏛️ Kiến trúc hệ thống

```text
┌──────────────────────────────────────────┐
│        React + Vite Dashboard            │
│          (chạy trên máy cục bộ)          │
└──────────────────────────────────────────┘
                    │ REST API
                    ▼
┌──────────────────────────────────────────┐
│        FastAPI Backend trên EC2          │
│         EBS · IAM Role · systemd         │
└──────────────────────────────────────────┘
          │                         │
          ▼                         ▼
 Amazon RDS for              Amazon CloudWatch
   PostgreSQL                 Logs · Metrics · Alarms
          ▲
          │ Telemetry · Polling · ACK
          ▼
┌──────────────────────────────────────────┐
│       YOLO UNO / Python Simulator        │
└──────────────────────────────────────────┘
```

### Thành phần chính

1. **YOLO UNO / Python Simulator:** Gửi telemetry, nhận lệnh đang chờ và phản hồi ACK. Phần cứng YOLO UNO là thiết bị chính; simulator hỗ trợ kiểm thử khi chưa kết nối phần cứng.
2. **Amazon EC2 và EBS:** Chạy backend FastAPI dưới dạng dịch vụ `systemd`; EBS là ổ đĩa gốc của EC2.
3. **Amazon RDS for PostgreSQL:** Lưu thiết bị, lịch sử telemetry và vòng đời lệnh.
4. **React + Vite Dashboard:** Hiển thị dữ liệu mới nhất, lịch sử và gửi yêu cầu điều khiển thiết bị.
5. **Amazon CloudWatch:** Thu thập log, theo dõi metric EC2/RDS và đánh giá các cảnh báo.
6. **Amazon VPC, Security Group và IAM Role:** Kiểm soát kết nối mạng và quyền gửi dữ liệu giám sát theo nguyên tắc đặc quyền tối thiểu.

Các dịch vụ đang dùng gồm **Amazon EC2, Amazon EBS, Amazon RDS for PostgreSQL, Amazon VPC, Security Group, AWS IAM Role, Amazon CloudWatch và CloudWatch Alarms**.

AWS IoT Core, Lambda, API Gateway, DynamoDB, S3, Auto Scaling và Amazon SQS chưa được triển khai trong phiên bản hiện tại.

---

## 🔄 Luồng hoạt động chính

- **Telemetry:** YOLO UNO đọc cảm biến → gửi HTTP POST đến FastAPI → backend lưu vào PostgreSQL → dashboard lấy dữ liệu mới nhất và lịch sử.
- **Lệnh điều khiển:** Người vận hành thao tác trên dashboard → FastAPI tạo lệnh ở trạng thái `Pending` → thiết bị thăm dò và thực thi → thiết bị gửi ACK → backend cập nhật trạng thái `Executed`.
- **Giám sát:** CloudWatch Agent gửi log và metric của hệ điều hành EC2; CloudWatch theo dõi thêm metric mặc định của EC2 và RDS.

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
- **AWS:** EC2, EBS, RDS, VPC, Security Group, IAM Role, CloudWatch và CloudWatch Alarms.
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
- Chỉ cho phép RDS nhận kết nối PostgreSQL từ Security Group của EC2.
- Giới hạn SSH theo địa chỉ IP quản trị.
- Không ghi cố định AWS access key trong mã nguồn.
- Dùng IAM Role cho quyền của CloudWatch Agent.

---

## 📄 Bản quyền

Copyright © 2026 Phạm Lê Minh Khôi – HCMUT. All rights reserved.
