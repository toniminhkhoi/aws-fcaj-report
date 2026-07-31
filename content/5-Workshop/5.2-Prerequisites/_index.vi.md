---
title: "Điều kiện tiên quyết"
date: "2026-07-28"
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

## Mục tiêu

Chuẩn bị tài khoản AWS, công cụ cục bộ, phần cứng, thông tin truy cập và kiến thức nền trước khi tạo tài nguyên có tính phí. Workshop dùng **Asia Pacific (Singapore), `ap-southeast-1`** làm khu vực mặc định; CloudFront/WAF là dịch vụ global, còn S3, ALB, ASG, EC2, RDS, VPC và CloudWatch phải bám đúng thiết kế khu vực đã ghi.

## Bước 1 - Xác minh quyền truy cập AWS

- Tài khoản AWS có quyền xem thông tin thanh toán và đã bật MFA.
- Quyền đặc quyền tối thiểu cho S3, CloudFront, WAF, Elastic Load Balancing, Auto Scaling, EC2/EBS, RDS, VPC/Security Group, thao tác gắn IAM Role, CloudWatch và alarm.
- Có quyền gán IAM Role đã được phê duyệt cho EC2 (`iam:PassRole`). Workshop không yêu cầu `AdministratorAccess`.
- Cặp khóa EC2 được lưu trên máy cục bộ; tuyệt đối không commit file `.pem` hoặc `.key`.
- Biết địa chỉ IP công khai của quản trị viên để tạo rule SSH: `<ADMIN_IP>/32`.

Trước khi cấp phát, ghi lại khu vực, CIDR của VPC, hai application subnet, các DB subnet theo Availability Zone cần thiết, tiền tố đặt tên tài nguyên và người chịu trách nhiệm dọn dẹp.

## Bước 2 - Xác minh công cụ cục bộ và phiên bản

Chạy từng lệnh trong đúng môi trường được chỉ định. Có thể dùng phiên bản mới hơn nếu mã nguồn của dự án hỗ trợ.

| Công cụ | Môi trường và lệnh kiểm tra | Kết quả mong đợi |
| :--- | :--- | :--- |
| Git | PowerShell: `git --version` | Hiển thị phiên bản Git |
| Python | PowerShell: `python --version` | Phiên bản tương thích với FastAPI `0.128.8` và SQLAlchemy `2.0.46` |
| Node.js | PowerShell: `node --version` | Phiên bản tương thích với Vite `8.1.1` và TypeScript `6.0.2` |
| npm | PowerShell: `npm --version` | Hiển thị phiên bản npm |
| PlatformIO | Terminal PlatformIO: `pio --version` | Hiển thị phiên bản PlatformIO Core |
| PostgreSQL client | PowerShell: `psql --version` | Hiển thị phiên bản PostgreSQL client |
| Trình duyệt | Mở DevTools → Network | Có thể quan sát và kiểm tra yêu cầu mạng |
| VS Code | Help → About | Hiển thị phiên bản đã cài đặt |

Nếu PowerShell không tìm thấy công cụ, mở lại terminal sau khi cài và chạy `Get-Command <tool>`. Không trộn `%USERPROFILE%` của Command Prompt với `$env:USERPROFILE` của PowerShell.

## Bước 3 - Chuẩn bị phần cứng và an toàn điện

Chuẩn bị:

- YOLO UNO / ESP32-S3 và cáp USB có khả năng truyền dữ liệu;
- cảm biến nhiệt độ, độ ẩm DHT20;
- cảm biến ánh sáng analog;
- module quạt và driver phù hợp khi cần;
- đèn hoặc module relay;
- động cơ servo điều khiển rèm;
- màn hình LCD1602 I2C có trong firmware cuối;
- dây nối, breadboard và bộ nguồn có thông số phù hợp.

Không cấp nguồn trực tiếp cho động cơ hoặc tải relay từ GPIO. Hãy kiểm tra mức điện áp, mạch điều khiển/diode bảo vệ và nối chung mass trước khi kết nối thiết bị chấp hành.

## Bước 4 - Xác minh mã nguồn và thông tin bí mật

Mã nguồn ứng dụng nằm trong kho riêng `aws-iot-dashboard`. Kho mã nguồn có `requirements.txt` của backend, `package.json` của frontend, `platformio.ini`, `boards/yolo_uno.json` và `include/secrets.example.h`. File `.gitignore` đã loại trừ `.env`, `.pem`, môi trường ảo, đầu ra biên dịch và `node_modules`; trước khi chia sẻ, cần xác nhận `hardware/include/secrets.h` vẫn chưa được Git theo dõi.

Qua rà soát mã nguồn, đã xác nhận điểm vào Uvicorn là `main:app`, tài khoản Amazon Linux là `ec2-user`, môi trường ảo là `venv`, ba bảng dữ liệu gồm `devices`, `telemetry_logs`, `commands`; sơ đồ GPIO được trình bày tại mục 5.6.

## Bước 5 - Xác nhận kiến thức tiên quyết

Người học nên hiểu các phương thức và mã trạng thái REST, truy vấn PostgreSQL, cách tham chiếu Security Group, FastAPI/OpenAPI, React/Vite, PlatformIO, quyền Linux cơ bản và `systemd`.

## Bước 6 - Xác nhận phạm vi công cụ tùy chọn

| Công cụ | Có bắt buộc cho cách triển khai hiện tại không? | Lý do |
| :--- | :---: | :--- |
| AWS Management Console | Có | Quy trình cấp phát tài nguyên và thu thập bằng chứng sử dụng Console |
| AWS CLI | Không | Hữu ích khi kiểm kê/xác minh, nhưng không có bước triển khai bắt buộc nào phụ thuộc vào CLI |
| AWS SAM CLI | Không | Dự án không triển khai Lambda hoặc ứng dụng SAM |
| AWS CDK | Không | Hạ tầng hiện tại không được định nghĩa bằng ứng dụng CDK |
| Terraform | Không | Mã nguồn đã rà soát không có cấu hình hoặc state Terraform |
| CloudFormation | Không | Workshop không tạo stack CloudFormation |

Sau này, người học có thể bổ sung Infrastructure as Code khi được phê duyệt; khi đó cần tài liệu hóa riêng danh sách tài nguyên và quy trình dọn dẹp. Ở phiên bản hiện tại, việc chưa cài AWS CLI, SAM, CDK hoặc Terraform không cản trở quá trình thực hành.

## Bước 7 - Kiểm tra mức độ sẵn sàng

- [ ] Đã thống nhất khu vực AWS và quy tắc đặt tên.
- [ ] Quyền AWS theo nguyên tắc đặc quyền tối thiểu hoạt động.
- [ ] Biết `<ADMIN_IP>` và vị trí EC2 key.
- [ ] Hoàn tất mọi lệnh kiểm tra version.
- [ ] Đã rà soát yêu cầu nguồn của phần cứng.
- [ ] Có kho mã nguồn và file mẫu chứa thông tin bí mật đã được loại khỏi Git.
- [ ] Đã phân công người dọn dẹp và nơi lưu bằng chứng.

![Phiên bản công cụ phát triển cục bộ đã xác minh](/images/5-Workshop/5.2-prerequisites/development-tools-versions.png)
*Hình 2. Bằng chứng terminal cho phiên bản Git, Python, Node.js, npm và PlatformIO được dùng trong dự án.*

## Kết quả mong đợi

Chỉ tiếp tục khi mọi điều kiện bắt buộc ở trên đã hoàn tất. Nếu thiếu kho mã nguồn hoặc sơ đồ chân firmware chính xác, hãy ghi nhận đây là vướng mắc trong bàn giao; không tự điền giá trị theo suy đoán và không cấp nguồn cho mạch.

## Xử lý sự cố

| Hiện tượng | Nội dung cần kiểm tra |
| :--- | :--- |
| Thao tác AWS bị từ chối | Danh tính đang đăng nhập, MFA, quyền dịch vụ cần thiết, `iam:PassRole` và đúng tài khoản |
| Không tìm thấy lệnh của công cụ | Đường dẫn cài đặt, mở lại terminal và chạy `Get-Command <tool>` |
| Kho mã nguồn thiếu thành phần | Đúng nhánh/commit và có đủ backend, frontend, hardware cùng file mẫu thông tin bí mật |
| File bí mật bị Git theo dõi | Dừng lại, bỏ file khỏi thay đổi, thay mới giá trị đã lộ và kiểm tra `.gitignore` |
| Chưa rõ yêu cầu phần cứng | Không cấp nguồn cho thiết bị chấp hành; xác minh sơ đồ chân trong firmware đang dùng và nguồn đúng định mức |

Tiếp theo: [xem kiến trúc và ranh giới dịch vụ](../5.3-Architecture-and-Service-Design/).
