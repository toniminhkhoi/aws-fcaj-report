---
title: "Bàn giao dự án"
date: "2026-07-28"
weight: 12
chapter: false
pre: " <b> 5.12. </b> "
---

## Tổng quan và mục tiêu

Bàn giao đầy đủ mã nguồn, cấu hình, tài liệu vận hành và bằng chứng để người tiếp quản có thể khởi động, xác minh, cập nhật, xử lý sự cố và dọn dẹp mô hình thử nghiệm một cách an toàn.

## Cấu trúc kho mã nguồn

Gói bàn giao ứng dụng cần có:

```text
<application-repository>/
├── backend/              # FastAPI, Pydantic, SQLAlchemy, requirements
├── frontend/             # React, Vite, TypeScript, Tailwind CSS
├── hardware/             # Firmware PlatformIO và định nghĩa bo mạch YOLO UNO
│   └── include/
│       └── secrets.example.h
└── README.md
```

Mã nguồn ứng dụng đã được rà soát và được quản lý riêng tại `F:\aws-iot-dashboard`; kho hiện tại chỉ chứa báo cáo Hugo và nội dung Workshop. Hồ sơ bàn giao cần ghi rõ:

- kho mã nguồn ứng dụng: `<SOURCE_REPOSITORY_URL>`;
- video minh họa: `<VIDEO_DEMO_URL>`;
- mã commit của phiên bản đang triển khai: `<COMMIT_SHA>`;
- nơi lưu danh mục tài nguyên AWS và bằng chứng bàn giao: `<HANDOVER_EVIDENCE_LOCATION>`.

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

## Thông tin bí mật cần lưu cục bộ

| Vị trí | Giá trị | Quy tắc lưu |
| :--- | :--- | :--- |
| Backend `.env` | `DATABASE_URL` và các thiết lập được định nghĩa trong mã nguồn | Chỉ lưu trên EC2 hoặc máy cục bộ; giới hạn quyền truy cập; loại khỏi Git |
| Firmware `secrets.h` | Wi-Fi, API URL, `room_01` | Chỉ lưu cục bộ; loại khỏi Git |
| Frontend `.env.local` nếu dùng | URL gốc của API | Chỉ lưu cục bộ; loại khỏi Git |
| Khóa EC2 | Khóa riêng tư | Lưu trong nơi quản lý bí mật đã được phê duyệt; không đưa vào Git |

Tài liệu bàn giao phải nêu cách cấp và xoay vòng thông tin bí mật. Không ghi thông tin xác thực dưới dạng văn bản thuần trong báo cáo.

## Danh sách kiểm tra AWS và vận hành

- [ ] Xác định đúng tài khoản AWS và Khu vực.
- [ ] Đã ghi lại VPC, public subnet, DB Subnet Group, route table và các thẻ tài nguyên.
- [ ] Đã ghi lại EC2, EBS, người giữ khóa, IAM Role, `iot-backend-sg` và `ec2-rds-1`.
- [ ] Đã ghi lại định danh/endpoint RDS, cơ sở dữ liệu `iot_dashboard` và `rds-ec2-1`.
- [ ] RDS vẫn ở private subnet; cổng 5432 chỉ nhận kết nối từ Security Group của EC2.
- [ ] `aws-iot-backend` và CloudWatch Agent tự chạy khi hệ điều hành khởi động.
- [ ] Đã ghi lại các log group của backend, namespace/dimension của metric, thời gian lưu log và các alarm.
- [ ] Đã ghi lại bản firmware cho `room_01`, sơ đồ chân GPIO chính xác và yêu cầu cấp nguồn an toàn.
- [ ] Đã liên kết kết quả T01–T15 mới nhất và các vấn đề còn mở.
- [ ] Đã chỉ định người chịu trách nhiệm chi phí và ngày dọn dẹp.

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

## Danh sách kiểm tra bàn giao cuối

- [ ] Người nhận mở được liên kết mã nguồn và xác định đúng mã commit.
- [ ] Git, hình ảnh, video và Workshop không chứa thông tin xác thực.
- [ ] Đã trình bày cách khởi động backend, frontend và firmware.
- [ ] Đã rà soát các route OpenAPI và lược đồ cơ sở dữ liệu dựa trên mã nguồn.
- [ ] Đã bàn giao sơ đồ chân GPIO và sơ đồ cấp nguồn.
- [ ] Ma trận kiểm thử có kết quả thực tế, bằng chứng và trạng thái.
- [ ] Bằng chứng đóng góp được gắn đúng thành viên và phần nhìn lại cá nhân đã được rà soát.
- [ ] Đã xác nhận cấu hình CloudWatch và ngưỡng cảnh báo.
- [ ] Đã xác nhận các vấn đề còn mở, hạn chế, người phụ trách, quyết định chi phí và trạng thái dọn dẹp.

<!-- TODO IMAGE: /images/5-Workshop/5.12-handover/repository-handover-checklist.png — Danh sách kiểm tra bàn giao kho mã nguồn, tài nguyên và kết quả kiểm thử; đã che thông tin nhạy cảm, có mã commit, người phụ trách, vấn đề còn mở và xác nhận của nhóm. -->

## Demo và tài nguyên bàn giao

Gói bàn giao cuối cùng gồm:

- Repository source code
- Video demo end-to-end
- Tài liệu triển khai và vận hành
- Workshop song ngữ
- Sơ đồ kiến trúc
- Hướng dẫn clean-up tài nguyên AWS

Xem đầy đủ tại [Tài liệu tham khảo]({{% relref "8-References/_index.vi.md" %}}).

Quay lại [trang Workshop](../).
