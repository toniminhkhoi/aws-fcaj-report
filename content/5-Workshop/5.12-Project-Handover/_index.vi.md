---
title: "Bàn giao dự án"
date: "2026-07-28"
weight: 12
chapter: false
pre: " <b> 5.12. </b> "
---

## Tổng quan và mục tiêu

Bàn giao đầy đủ mã nguồn, cấu hình, tài liệu vận hành và bằng chứng để người tiếp quản có thể khởi động, xác minh, cập nhật, xử lý sự cố và dọn dẹp mô hình thử nghiệm một cách an toàn.

## Cấu trúc repository bàn giao

Repository chính chứa source code backend, frontend, firmware YOLO UNO, sơ đồ kiến trúc và tài liệu README song ngữ. Branch `main` được sử dụng làm phiên bản bàn giao cuối cùng của project.

<p align="center">
  <img src="/images/5-Workshop/5.12-handover/repository-handover-checklist.png"
       alt="Cấu trúc GitHub repository của dự án AWS IoT Monitoring and Control Dashboard"
       width="100%" />
</p>

*Hình 22. Cấu trúc repository cuối cùng của dự án, bao gồm backend, frontend, firmware YOLO UNO, sơ đồ kiến trúc và tài liệu README song ngữ.*

Ảnh cho thấy repository bàn giao có source code cho các thành phần chính của hệ thống, tài liệu README bằng hai ngôn ngữ và thư mục lưu sơ đồ. Những file chứa cấu hình bí mật phải được loại trừ bằng `.gitignore` và kiểm tra riêng trước khi bàn giao.

## Quy trình khởi động

Backend trên EC2 Linux Bash:

```bash
sudo systemctl start aws-iot-backend
sudo systemctl status aws-iot-backend --no-pager
curl -i http://127.0.0.1:8000/api/health
```

Frontend trên Windows PowerShell:

```powershell
Set-Location .\aws-iot-dashboard\frontend
npm install
npm run dev
```

Phần cứng trong terminal của PlatformIO:

```bash
pio run
pio run --target upload
pio device monitor --baud 115200
```

## Quy trình cập nhật phiên bản triển khai

Trong EC2 Linux Bash:

```bash
cd ~/aws-iot-dashboard
git status --short
git pull --ff-only
source backend/venv/bin/activate
pip install -r backend/requirements.txt
cd backend
python -m app.database.init_db
cd ..
sudo systemctl restart aws-iot-backend
sudo systemctl status aws-iot-backend --no-pager
curl -i http://127.0.0.1:8000/api/health
```

Trước khi cập nhật, cần rà soát các thay đổi về mô hình dữ liệu, lược đồ và ghi chú phát hành. `app.database.init_db` sử dụng `create_all` của SQLAlchemy, không phải công cụ quản lý migration; mọi thay đổi có nguy cơ làm mất dữ liệu hoặc gây mất tương thích phải tuân theo một quy trình riêng đã được phê duyệt. Hãy ghi lại mã commit trước và sau khi cập nhật, kèm phương án quay lui. Không dùng `git reset --hard` để xóa thay đổi cục bộ.

## Kiểm tra cơ sở dữ liệu và CloudWatch

Từ EC2 Linux Bash:

```bash
psql "host=<RDS_ENDPOINT> port=5432 dbname=iot_dashboard user=<DB_USER> sslmode=require"
sudo systemctl status amazon-cloudwatch-agent --no-pager
sudo tail -n 100 /opt/aws/amazon-cloudwatch-agent/logs/amazon-cloudwatch-agent.log
```

Trong `psql`, chạy `\dt`, kiểm tra các bảng `devices`, `telemetry_logs`, `commands` và dùng truy vấn chỉ đọc để xác minh dữ liệu. Trong CloudWatch, hãy chọn đúng khu vực, kiểm tra các log group đã cấu hình, thời điểm có log gần nhất, metric tùy chỉnh `IoTDashboard/EC2`, metric CPU mặc định của EC2, CPU/số kết nối của RDS, cùng tên và trạng thái của năm alarm đã mô tả.

## Hạn chế đã biết

Mô hình thử nghiệm hiện chỉ phục vụ một phòng và dùng HTTP trực tiếp qua cổng 8000 để minh họa. Địa chỉ IP công khai của EC2 có thể thay đổi; thiết bị kiểm tra lệnh theo chu kỳ; giá trị ánh sáng analog chưa được hiệu chuẩn. Hệ thống chưa triển khai HTTPS, xác thực route hoặc API key, High Availability, Multi-AZ, Load Balancer, giới hạn tần suất gọi API hay mô hình AI. Frontend vẫn có thể chuyển sang dữ liệu mô phỏng, báo thành công giả sau lỗi, lưu chế độ trên máy người dùng, gọi giá trị ánh sáng là Lux và ghi trực tiếp địa chỉ EC2. Backend chưa kiểm tra chặt danh sách lệnh hợp lệ và quyền sở hữu khi ACK. Các thông tin về GPIO, đường dẫn/lược đồ mã nguồn và ngưỡng alarm đã được ghi ở 5.5, 5.6 và 5.9, nhưng vẫn cần đối chiếu với bằng chứng từ môi trường đang chạy.

## Trách nhiệm nhóm

| Thành viên | Trách nhiệm | Bằng chứng đóng góp |
| :--- | :--- | :--- |
| **Phạm Lê Minh Khôi** | Kiến trúc AWS; VPC, Security Group, IAM Role, EC2, RDS và CloudWatch; DevOps; phần cứng YOLO UNO; cảm biến, thiết bị chấp hành, telemetry, cơ chế thăm dò lệnh và ACK | [5.3 kiến trúc](../5.3-Architecture-and-Service-Design/), [5.4 AWS](../5.4-AWS-Infrastructure-Setup/), [5.6 phần cứng](../5.6-Hardware-Integration/), [5.9 giám sát](../5.9-CloudWatch-Monitoring/) |
| **Ngô Minh Thuận** | Backend FastAPI; endpoint, lược đồ Pydantic, mô hình SQLAlchemy, tích hợp PostgreSQL, xử lý telemetry, vòng đời lệnh và ACK | [5.3 API/dữ liệu](../5.3-Architecture-and-Service-Design/), [5.5 backend/cơ sở dữ liệu](../5.5-Backend-and-Database/), [5.8 kiểm thử](../5.8-End-to-End-Testing/) |
| **Thượng Đình Hưng** | Frontend React + Vite; giao diện dashboard, trực quan hóa telemetry, chức năng điều khiển, tích hợp tổng thể, xử lý lỗi và quay/dựng video minh họa | [5.7 frontend](../5.7-Frontend-Integration/), [5.8 bằng chứng tích hợp](../5.8-End-to-End-Testing/), liên kết video trong phần cấu trúc kho mã nguồn |
| **Lê Bảo Khánh** | Tài liệu, proposal, blog, nhật ký công việc hằng tuần, báo cáo sự kiện, Workshop, rà soát song ngữ, điều hướng, ảnh chụp màn hình và bảo đảm chất lượng | [5.1 tiêu chí/kết quả](../5.1-Workshop-overview/), [5.11 tài liệu/điều chỉnh](../5.11-Results-Challenges-Future/), Workshop song ngữ và kết quả kiểm tra Hugo |

Bảng trên giữ nguyên nội dung phân công đã thống nhất và dẫn người chấm tới bằng chứng đóng góp tương ứng. Bảng này không thay thế [phần nhìn lại riêng của từng thành viên ở mục 5.11](../5.11-Results-Challenges-Future/). Trước khi nộp, mỗi thành viên cần rà soát và xác nhận cả phạm vi phụ trách lẫn phần nhìn lại của mình.

## Checklist bàn giao cuối cùng

| Hạng mục | Trạng thái |
|---|---|
| Source code FastAPI backend | Hoàn thành |
| Source code React + Vite frontend | Hoàn thành |
| Firmware PlatformIO cho YOLO UNO | Hoàn thành |
| README tiếng Anh và tiếng Việt | Hoàn thành |
| Sơ đồ kiến trúc và tài liệu đấu nối phần cứng | Hoàn thành |
| Video demo end-to-end | Hoàn thành |
| Tài liệu Workshop song ngữ | Hoàn thành |
| Hướng dẫn triển khai backend và phần cứng | Hoàn thành |
| Hướng dẫn kiểm thử và CloudWatch | Hoàn thành |
| Hướng dẫn clean-up tài nguyên AWS | Hoàn thành |
| Thông tin bí mật không được commit lên Git | Đã kiểm tra |

### Kiểm tra thông tin bí mật

Thông tin bí mật, còn gọi là secrets hoặc credentials, là các dữ liệu dùng để xác thực và kết nối hệ thống. Các giá trị này không được commit lên GitHub repository.

Các file và thông tin cần được loại trừ gồm:

- `backend/.env`
- `hardware/include/secrets.h`
- Wi-Fi SSID nếu không muốn công khai
- Mật khẩu Wi-Fi
- Amazon RDS username và password
- Chuỗi `DATABASE_URL` có chứa credential
- AWS Access Key ID
- AWS Secret Access Key
- AWS session token
- SSH private key dạng `.pem`
- API token
- GitHub personal access token
- Các credential cá nhân khác

Các file mẫu được phép commit:

- `.env.example`
- `secrets.example.h`

Tuy nhiên, các file mẫu chỉ được chứa placeholder, ví dụ:

```env
DATABASE_URL=postgresql://USERNAME:PASSWORD@RDS_ENDPOINT:5432/DATABASE_NAME
```

```cpp
#define WIFI_SSID "YOUR_WIFI_SSID"
#define WIFI_PASSWORD "YOUR_WIFI_PASSWORD"
#define API_BASE_URL "http://YOUR_EC2_ADDRESS"
```

Không được đưa giá trị thật vào các file mẫu.

> Việc xóa secret khỏi phiên bản hiện tại chưa chắc đã xóa secret khỏi lịch sử Git. Nếu secret từng được commit, cần thu hồi hoặc thay đổi credential đó và làm sạch lịch sử repository khi cần thiết.

### Kiểm tra trước khi bàn giao

Chạy các lệnh sau tại thư mục gốc của repository.

Kiểm tra trạng thái working tree:

```bash
git status
```

Liệt kê toàn bộ file đang được Git theo dõi:

```bash
git ls-files
```

Tìm các file nhạy cảm trong lịch sử commit:

```bash
git log --all --oneline -- .env secrets.h "*.pem"
```

Tìm tên biến thường dùng cho credential trong các file đang được theo dõi:

```bash
git grep -n -I -E "AWS_ACCESS_KEY_ID|AWS_SECRET_ACCESS_KEY|AWS_SESSION_TOKEN|DATABASE_URL|WIFI_PASSWORD|PRIVATE_KEY|API_TOKEN"
```

Kiểm tra riêng các file không nên được Git theo dõi:

```bash
git ls-files | grep -E '(^|/)\.env$|secrets\.h$|\.pem$'
```

Lưu ý:

- Không chèn output có chứa password hoặc credential vào Workshop.
- `git status` chỉ kiểm tra trạng thái hiện tại.
- `git log --all` dùng để kiểm tra lịch sử commit.
- Nếu phát hiện AWS key, password hoặc token từng bị commit, phải thu hồi hoặc thay đổi credential đó.
- Không chỉ xóa file rồi tiếp tục sử dụng lại credential cũ.

## Tài nguyên bàn giao

Gói bàn giao cuối cùng của dự án gồm:

- GitHub repository chứa source code.
- Video demo end-to-end.
- Tài liệu triển khai backend.
- Hướng dẫn firmware YOLO UNO.
- Workshop song ngữ.
- Sơ đồ kiến trúc AWS.
- Hướng dẫn kiểm thử và giám sát CloudWatch.
- Hướng dẫn clean-up tài nguyên AWS.

Source code, video demo và các tài liệu liên quan được tổng hợp trong mục [Tài liệu tham khảo]({{% relref "8-References/_index.vi.md" %}}).

Quay lại [trang Workshop](../).
