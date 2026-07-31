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

Xác minh backend trên một instance của ASG (EC2 Linux Bash):

```bash
sudo systemctl start aws-iot-backend
sudo systemctl status aws-iot-backend --no-pager
curl -i http://127.0.0.1:8000/api/health
```

Xác minh dịch vụ từ bên ngoài:

```powershell
curl.exe -sS -i "http://<ALB_DNS_NAME>/api/health"
curl.exe -sS -i "https://<CLOUDFRONT_DOMAIN>/api/health"
```

Luồng phát triển frontend cục bộ trên Windows PowerShell:

```powershell
Set-Location .\aws-iot-dashboard\frontend
npm install
npm run dev
```

Ở production, tạo bản build, đồng bộ thư mục `dist` lên S3 private, tạo CloudFront invalidation và xác minh default behavior tới S3 cùng behavior `/api/*` tới ALB:

```powershell
Set-Location .\aws-iot-dashboard\frontend
npm install
npm run build
aws s3 sync dist "s3://<FRONTEND_BUCKET>" --delete
aws cloudfront create-invalidation `
  --distribution-id "<CLOUDFRONT_DISTRIBUTION_ID>" `
  --paths "/*"
```

Chỉ tạo invalidation khi nội dung frontend thay đổi để tránh thao tác và chi phí không cần thiết.

Phần cứng trong terminal của PlatformIO:

```bash
pio run -e yolo_uno
pio run -e yolo_uno --target upload
pio device monitor --baud 115200
```

## Quy trình cập nhật phiên bản triển khai

Luồng phát hành của ASG theo hướng immutable: xác minh thay đổi trên instance nguồn, tạo AMI/Launch Template version mới, cập nhật ASG, chạy instance refresh và kiểm tra target health trước khi bỏ phiên bản cũ. Các lệnh dưới đây vẫn được giữ làm quy trình xác minh trên instance nguồn:

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

Sau khi xác minh:

1. Tạo AMI backend và Launch Template version có đánh số.
2. Cập nhật `iot-backend-asg` sang template version mới.
3. Chạy instance refresh và giữ AMI/template cũ để rollback.
4. Xác nhận desired capacity 2, cả hai target Healthy, ALB health trả HTTP 200 và có log CloudWatch mới.
5. Build/sync frontend lên S3 private và chỉ invalidate CloudFront khi frontend thay đổi.

## Kiểm tra cơ sở dữ liệu và CloudWatch

Từ EC2 Linux Bash:

```bash
psql "host=<RDS_ENDPOINT> port=5432 dbname=iot_dashboard user=<DB_USER> sslmode=require"
sudo systemctl status amazon-cloudwatch-agent --no-pager
sudo tail -n 100 /opt/aws/amazon-cloudwatch-agent/logs/amazon-cloudwatch-agent.log
```

Trong `psql`, chạy `\dt`, kiểm tra các bảng `devices`, `telemetry_logs`, `commands` và dùng truy vấn chỉ đọc để xác minh dữ liệu. Trong CloudWatch, chọn đúng khu vực, kiểm tra log stream của cả hai instance ASG, timestamp gần nhất, guest metrics, widget ALB/ASG/EC2/RDS và đủ tám alarm đã mô tả.

## Hạn chế đã biết

Hệ thống hiện vẫn chỉ phục vụ một phòng, dùng REST polling và báo giá trị ánh sáng analog chưa hiệu chuẩn. CloudFront cung cấp HTTPS phía viewer, nhưng ALB origin và route trực tiếp của thiết bị dùng HTTP; API chưa có xác thực mạnh hoặc rate limiting, còn WAF managed rules vẫn ở Count/Monitor. ALB/ASG và RDS Multi-AZ đã triển khai nhưng chưa có bằng chứng diễn tập failover có kiểm soát. Frontend vẫn có thể dùng dữ liệu mô phỏng hoặc báo thành công sau lỗi, lưu mode cục bộ và gắn nhãn Lux cho giá trị chưa hiệu chuẩn; backend chưa kiểm tra chặt enum lệnh và quyền sở hữu ACK. Panel đề xuất là luật xác định trước, không phải mô hình AI/ML.

## Trách nhiệm nhóm

| Thành viên | Trách nhiệm | Bằng chứng đóng góp |
| :--- | :--- | :--- |
| **Phạm Lê Minh Khôi** | Kiến trúc AWS; VPC, Security Group, IAM Role, EC2, RDS và CloudWatch; DevOps; phần cứng YOLO UNO; cảm biến, thiết bị chấp hành, telemetry, cơ chế thăm dò lệnh và ACK | [5.3 kiến trúc](../5.3-Architecture-and-Service-Design/), [5.4 AWS](../5.4-AWS-Infrastructure-Setup/), [5.6 phần cứng](../5.6-Hardware-Integration/), [5.9 giám sát](../5.9-CloudWatch-Monitoring/) |
| **Ngô Minh Thuận** | Backend FastAPI; endpoint, lược đồ Pydantic, mô hình SQLAlchemy, tích hợp PostgreSQL, xử lý telemetry, vòng đời lệnh và ACK | [5.3 API/dữ liệu](../5.3-Architecture-and-Service-Design/), [5.5 backend/cơ sở dữ liệu](../5.5-Backend-and-Database/), [5.8 kiểm thử](../5.8-End-to-End-Testing/) |
| **Thượng Đình Hưng** | Frontend React + Vite; giao diện dashboard, trực quan hóa telemetry, chức năng điều khiển, tích hợp tổng thể, xử lý lỗi và quay/dựng video minh họa | [5.7 frontend](../5.7-Frontend-Integration/), [5.8 bằng chứng tích hợp](../5.8-End-to-End-Testing/), [video demo]({{% relref "8-References/8.2-demo-video/_index.vi.md" %}}) |
| **Lê Bảo Khánh** | Tài liệu, proposal, blog, nhật ký công việc hằng tuần, báo cáo sự kiện, Workshop, rà soát song ngữ, điều hướng, ảnh chụp màn hình và bảo đảm chất lượng | [5.1 tiêu chí/kết quả](../5.1-Workshop-overview/), [5.11 tài liệu/điều chỉnh](../5.11-Results-Challenges-Future/), Workshop song ngữ và các liên kết nội bộ |

Bảng trên tóm tắt phạm vi đóng góp đang được ghi nhận trong Workshop và dẫn tới các bằng chứng liên quan. Bảng này không thay thế [phần nhìn lại riêng của từng thành viên ở mục 5.11](../5.11-Results-Challenges-Future/). Trước khi nộp, mỗi thành viên cần rà soát và xác nhận phạm vi phụ trách cũng như phần nhìn lại của mình.

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
| Hướng dẫn triển khai backend, frontend và phần cứng | Hoàn thành |
| Hướng dẫn kiểm thử và CloudWatch | Hoàn thành |
| Bằng chứng vận hành CloudFront/WAF/S3, ALB/ASG và RDS Multi-AZ | Hoàn thành |
| Hướng dẫn clean-up tài nguyên AWS | Hoàn thành |
| File secret thật không được Git theo dõi | Đã kiểm tra |
| Giá trị credential và lịch sử Git | Cần rà soát lần cuối trước khi nộp |

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
#define API_BASE_URL "http://YOUR_ALB_DNS_NAME"
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

Nếu lệnh không trả về kết quả, các file nhạy cảm nêu trên không được theo dõi trong phiên bản hiện tại. Kết quả này không thay thế việc kiểm tra giá trị credential trong các file cấu hình, mã nguồn và lịch sử Git.

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
- Tài liệu build và triển khai frontend.
- Hướng dẫn firmware YOLO UNO.
- Workshop song ngữ.
- Sơ đồ kiến trúc AWS.
- Hướng dẫn kiểm thử và giám sát CloudWatch.
- Hướng dẫn clean-up tài nguyên AWS.

Source code, video demo và các tài liệu liên quan được tổng hợp trong mục [Tài liệu tham khảo]({{% relref "8-References/_index.vi.md" %}}).

Quay lại [trang Workshop](../).
