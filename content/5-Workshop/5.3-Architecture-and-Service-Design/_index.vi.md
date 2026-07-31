---
title: "Kiến trúc và thiết kế dịch vụ"
date: "2026-07-28"
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

## Kiến trúc

![Kiến trúc AWS IoT Monitoring and Control Dashboard](/images/2-Proposal/IoT_Dashboard_Architecture.png)

*Hình 5-2. Kiến trúc hiện tại gồm CloudFront/WAF với S3 private origin, `/api/*` chuyển đến ALB, hai FastAPI instance trong ASG, RDS PostgreSQL Multi-AZ và tuyến thiết bị đi trực tiếp đến ALB.*

Người dùng dashboard và YOLO UNO nằm ngoài AWS. CloudFront phân phối bản build React + Vite từ S3 private và chuyển request browser `/api/*` đến ALB Internet-facing. YOLO UNO dùng trực tiếp DNS ALB. Trong VPC, ALB định tuyến đến hai FastAPI instance do ASG quản lý tại `ap-southeast-1a` và `ap-southeast-1c`; RDS PostgreSQL Multi-AZ có primary tại `ap-southeast-1c` và standby tại `ap-southeast-1b`. Mỗi backend instance có EBS root volume được mã hóa.

![Luồng dữ liệu trình duyệt, thiết bị, command, ACK và giám sát](/images/5-Workshop/5.3-architecture/architecture-data-flows.svg)

*Hình 5-3. Trình duyệt dùng CloudFront, còn YOLO UNO gọi trực tiếp ALB; hai tuyến cùng hội tụ tại mô hình telemetry và command của FastAPI/RDS.*

## Thành phần và lý do chọn dịch vụ AWS

| Thành phần/dịch vụ | Trách nhiệm và lý do |
| :--- | :--- |
| React + Vite + TypeScript + Tailwind CSS | Giao diện người vận hành được build lên S3 private và phân phối qua CloudFront |
| Amazon S3 + CloudFront + AWS WAF | Static origin private, phân phối HTTPS, định tuyến `/api/*` và theo dõi managed rule |
| Application Load Balancer + Target Group | Endpoint API ổn định và health check `/api/health` cho backend target |
| EC2 Auto Scaling Group | Duy trì hai FastAPI/Uvicorn instance và có thể scale từ 2 đến 4 |
| Amazon EC2 | Toàn quyền cấu hình FastAPI, Python, Uvicorn và `systemd` trên từng backend instance |
| Amazon EBS | Ổ đĩa gốc mã hóa, gắn với từng EC2 instance |
| Amazon RDS for PostgreSQL Multi-AZ | Lưu dữ liệu quan hệ có primary/standby phục vụ failover |
| Amazon VPC và subnet | Ranh giới mạng cho ALB, ASG instance và DB Subnet Group |
| Security Group | Tường lửa có trạng thái cho ALB-to-backend và backend-to-RDS |
| AWS IAM Role | Cấp quyền tạm thời để EC2 gửi dữ liệu giám sát |
| CloudWatch Agent | Phần mềm trên EC2 thu thập metric của hệ điều hành khách và file log |
| Amazon CloudWatch/Alarms | Lưu metric/log và đánh giá các ngưỡng |
| YOLO UNO / ESP32-S3 | Đọc cảm biến, điều khiển thiết bị chấp hành, thăm dò lệnh và gửi ACK |

IAM Role có nhiệm vụ cấp quyền; nó không phải CloudWatch Agent. Agent là một tiến trình được cài đặt và quản lý trên EC2.

## 5.3.3 Lựa chọn dịch vụ AWS và các đánh đổi kiến trúc

Dự án lựa chọn các dịch vụ AWS dựa trên bốn tiêu chí chính:

1. Phù hợp với kiến trúc và mã nguồn hiện tại.
2. Đơn giản để triển khai và giải thích trong Workshop.
3. Có thể giám sát, kiểm thử và vận hành trực tiếp.
4. Có chi phí hợp lý cho môi trường học tập và trình diễn.

Không phải dịch vụ serverless nào cũng cần thiết cho bài toán này. Hệ thống hiện chạy FastAPI liên tục, kết nối PostgreSQL và giao tiếp với YOLO UNO qua REST API trên HTTP. Vì vậy, nhóm chọn Amazon EC2 và Amazon RDS thay vì thiết kế lại toàn bộ hệ thống theo Lambda, API Gateway và DynamoDB.

### Các dịch vụ được lựa chọn

| Dịch vụ AWS | Vai trò trong hệ thống | Lý do lựa chọn | Đánh đổi |
| :--- | :--- | :--- | :--- |
| **Amazon EC2** | Chạy backend FastAPI, Uvicorn và CloudWatch Agent | Cho phép chủ động cấu hình môi trường Python, thư viện phụ thuộc, cổng mạng, dịch vụ `systemd` và file log | Nhóm phải tự quản lý hệ điều hành, cập nhật gói phần mềm, dịch vụ và một phần cấu hình bảo mật |
| **Amazon EBS** | Ổ đĩa gốc của EC2 | Cung cấp nơi lưu trữ bền vững cho hệ điều hành, mã nguồn, môi trường ảo và log cục bộ | Ổ đĩa không còn gắn vẫn có thể phát sinh chi phí nếu không được xóa |
| **Amazon RDS for PostgreSQL** | Lưu telemetry và trạng thái lệnh | PostgreSQL phù hợp với dữ liệu có cấu trúc và quan hệ giữa thiết bị, telemetry, lệnh; RDS giảm công việc quản trị so với tự cài PostgreSQL trên EC2 | Một cơ sở dữ liệu chạy liên tục có thể tốn hơn một số giải pháp serverless khi lưu lượng rất thấp |
| **Amazon VPC** | Cung cấp môi trường mạng cho EC2 và RDS | Cho phép kiểm soát subnet, định tuyến và kết nối giữa backend với cơ sở dữ liệu | Đòi hỏi cấu hình mạng và Security Group chính xác |
| **Security Groups** | Kiểm soát lưu lượng vào EC2 và RDS | Giới hạn SSH theo IP quản trị và chỉ cho phép RDS nhận cổng 5432 từ EC2 Security Group | Cấu hình sai có thể chặn kết nối cơ sở dữ liệu hoặc vô tình mở dịch vụ quá rộng |
| **AWS IAM Role** | Cấp quyền để EC2 gửi log và metric | Tránh lưu AWS access key trực tiếp trên EC2 hoặc trong mã nguồn | Policy phải tuân theo nguyên tắc đặc quyền tối thiểu |
| **Amazon CloudWatch** | Thu thập log và metric từ EC2, backend, RDS | Tập trung dữ liệu vận hành để quan sát và xử lý sự cố | Việc thu nhận log, thời gian lưu giữ và metric tùy chỉnh có thể phát sinh chi phí |
| **CloudWatch Alarms** | Theo dõi CPU, bộ nhớ, ổ đĩa và số kết nối cơ sở dữ liệu | Cảnh báo khi metric vượt ngưỡng vận hành đã cấu hình | Alarm chỉ hữu ích khi ngưỡng và khoảng thời gian đánh giá được đặt phù hợp |
| **Amazon S3 và CloudFront** | Lưu/phân phối frontend và định tuyến `/api/*` đến ALB | Giữ S3 origin private qua OAC và cung cấp endpoint HTTPS cho browser | Thay đổi cache/behavior cần quy trình deploy và invalidation có kiểm soát |
| **AWS WAF** | Áp dụng ba managed rule group ở CloudFront edge | Quan sát các web threat phổ biến trước khi bật enforcement | Rule hiện chạy Count/Monitor, chưa Block |
| **Application Load Balancer và Auto Scaling** | Phân phối API đến hai FastAPI target healthy | Loại bỏ single backend endpoint và duy trì desired capacity qua hai AZ | Phát sinh chi phí ALB/EC2 và cần quản lý vòng đời launch template |

### Vì sao chọn Amazon EC2 cho FastAPI backend?

FastAPI backend của dự án là một ứng dụng chạy liên tục và cung cấp nhiều REST API cho frontend và thiết bị YOLO UNO. Amazon EC2 phù hợp vì nhóm có thể:

- Cài đặt phiên bản Python và các thư viện phụ thuộc cần thiết.
- Chạy Uvicorn dưới dạng dịch vụ `systemd`.
- Chủ động cấu hình cổng `8000`.
- Cài CloudWatch Agent.
- Truy cập log và kiểm tra trạng thái dịch vụ qua SSH.
- Kết nối trực tiếp đến Amazon RDS for PostgreSQL.
- Gỡ lỗi toàn bộ yêu cầu telemetry, lệnh và ACK.

Amazon EC2 cũng giúp Workshop dễ trình bày hơn vì người học có thể quan sát rõ quá trình cài đặt backend, khởi động service, kiểm tra log và xử lý lỗi.

Đánh đổi chính là EC2 không phải dịch vụ được quản lý hoàn toàn. Nhóm vẫn phải cập nhật hệ điều hành, quản lý dịch vụ, kiểm tra dung lượng lưu trữ và bảo vệ SSH cũng như cổng backend.

### Vì sao chọn Amazon RDS for PostgreSQL?

Dữ liệu của dự án có cấu trúc và quan hệ rõ ràng:

- Một thiết bị gửi nhiều bản ghi telemetry.
- Một thiết bị có thể nhận nhiều lệnh.
- Mỗi lệnh có trạng thái như `Pending` hoặc `Executed`.
- Backend cần truy vấn dữ liệu theo thời gian và ID thiết bị.

PostgreSQL phù hợp với các truy vấn có cấu trúc, việc kiểm tra trạng thái lệnh và lưu lịch sử telemetry. Amazon RDS giảm công việc quản trị so với tự cài PostgreSQL trên EC2, từ khởi tạo cơ sở dữ liệu, quản lý dung lượng đến cung cấp metric tích hợp với CloudWatch.

Đánh đổi là RDS instance vẫn phát sinh chi phí theo thời gian hoạt động, kể cả khi lưu lượng của Workshop thấp.

### Vì sao chọn Amazon EBS, VPC và IAM Role?

- **Amazon EBS** cung cấp ổ đĩa gốc cho EC2, lưu bền vững hệ điều hành, mã nguồn đã lấy về, môi trường ảo Python và log backend cục bộ qua các lần khởi động lại thông thường. Dung lượng, snapshot, mã hóa và chi phí của ổ đĩa không còn gắn vẫn là trách nhiệm vận hành.
- **Amazon VPC** tạo ranh giới rõ ràng bằng subnet, route và Security Group. Thiết kế cho phép truy cập EC2 phục vụ demo trong khi RDS vẫn nằm riêng trong DB Subnet Group. Đổi lại, nhóm phải cấu hình cẩn thận và lưu bằng chứng đầy đủ.
- **AWS IAM Role** cấp thông tin xác thực tạm thời để EC2 gửi dữ liệu lên CloudWatch mà không lưu AWS access key trong mã nguồn hoặc `.env`. Role phải được giới hạn theo hành động và tài nguyên cần thiết; role không thay thế kiểm soát mạng hoặc CloudWatch Agent.

### Vì sao chọn Amazon CloudWatch?

CloudWatch được lựa chọn vì có thể tập trung:

- EC2 `CPUUtilization`.
- Memory và disk metric từ CloudWatch Agent.
- FastAPI backend logs.
- RDS `CPUUtilization`.
- RDS `DatabaseConnections`.
- Trạng thái của CloudWatch Alarms.

Nhờ đó, nhóm có thể chứng minh hệ thống không chỉ hoạt động đúng chức năng mà còn có khả năng giám sát và hỗ trợ xử lý sự cố.

### Vì sao chọn HTTP REST thay vì MQTT?

HTTP REST được chọn cho prototype `room_01` vì dự án đã sử dụng các endpoint FastAPI cho telemetry, thăm dò lệnh và ACK. Cùng một hợp đồng JSON có thể được kiểm tra trực tiếp bằng `curl`, PowerShell, browser DevTools, log Uvicorn và truy vấn PostgreSQL. Nhờ đó, từng bước của luồng đầu cuối dễ trình diễn và xử lý sự cố mà chưa cần bổ sung MQTT broker, topic, subscription, chứng chỉ thiết bị và các thành phần xử lý message riêng.

Đây là quyết định đánh đổi giữa phạm vi và độ đơn giản, không phải khẳng định HTTP tốt hơn MQTT cho mọi hệ thống IoT:

| Tiêu chí | HTTP REST trong prototype hiện tại | MQTT / AWS IoT Core khi mở rộng |
| :--- | :--- | :--- |
| Tích hợp | Tái sử dụng API request/response hiện có của FastAPI | Cần broker topic, policy, certificate và subscriber |
| Kiểm chứng | Dễ đối chiếu HTTP status, JSON response, API log và bản ghi database | Cần kiểm tra publish/subscribe và bằng chứng phía broker |
| Phân phối lệnh | Thiết bị định kỳ thăm dò command `Pending` trong PostgreSQL rồi gửi ACK | Broker có thể đẩy lệnh qua topic mà thiết bị đã subscribe |
| Kết nối và băng thông | Polling lặp lại tạo nhiều request và protocol overhead hơn | Kết nối duy trì, nhẹ thường hiệu quả hơn khi có nhiều thiết bị |
| Đảm bảo chuyển phát | Ứng dụng tự cài đặt retry, trạng thái command và ACK | MQTT có các mức QoS và session feature dành cho messaging |
| Mức độ phù hợp hiện tại | Đủ cho một phòng mô hình với tải telemetry/command nhỏ | Phù hợp hơn cho nhiều thiết bị, kết nối gián đoạn, băng thông thấp hoặc event-driven delivery |

Khi mở rộng, cần đánh giá AWS IoT Core với MQTT cùng định danh riêng cho từng thiết bị, xoay vòng certificate, phân quyền topic, QoS, hành vi offline, chi phí và kế hoạch chuyển đổi vòng đời command REST hiện tại. MQTT vẫn là hướng phát triển tương lai và không được mô tả là đã triển khai trong Workshop này.

### Các dịch vụ không được sử dụng trong phiên bản hiện tại

| Dịch vụ | Lý do chưa lựa chọn |
| :--- | :--- |
| **AWS Lambda** | Backend FastAPI hiện chạy liên tục dưới dạng dịch vụ trên EC2. Chuyển sang Lambda đòi hỏi thay đổi mô hình triển khai, vòng đời xử lý và cách kết nối cơ sở dữ liệu |
| **Amazon API Gateway** | API browser dùng CloudFront `/api/*` đến ALB, còn YOLO UNO gọi ALB trực tiếp. API Gateway chưa được triển khai |
| **Amazon DynamoDB** | Dữ liệu được thiết kế theo mô hình quan hệ; backend đã dùng SQLAlchemy với PostgreSQL |
| **AWS IoT Core** | YOLO UNO hiện giao tiếp trực tiếp với FastAPI bằng REST API qua HTTP. MQTT và chứng chỉ thiết bị là các lựa chọn có thể xem xét sau này |
| **Amazon SQS** | Luồng lệnh hiện dùng bản ghi `Pending` trong PostgreSQL và cơ chế thăm dò của thiết bị; chưa triển khai hàng đợi, bên gửi hoặc bên nhận SQS |

Việc chưa sử dụng các dịch vụ trên không có nghĩa chúng không phù hợp với IoT. Đây là quyết định giới hạn phạm vi, giúp nhóm tập trung vào luồng đầu cuối giữa phần cứng, REST API, PostgreSQL, dashboard và CloudWatch.

### Đánh giá về chi phí và độ đơn giản

Kiến trúc hiện tại ưu tiên khả năng quan sát và triển khai trực tiếp hơn là tối ưu hoàn toàn theo mô hình serverless.

- **Độ đơn giản:** EC2 cho phép chạy nguyên backend FastAPI mà không phải tách thành nhiều hàm Lambda.
- **Dịch vụ được quản lý:** RDS giảm công việc quản trị so với tự cài PostgreSQL trên EC2.
- **Chi phí:** CloudFront/WAF, S3, ALB, hai EC2 với EBS, RDS Multi-AZ và CloudWatch có thể phát sinh phí và phải nằm trong kế hoạch dọn dẹp.
- **Giá trị học tập:** Người học có thể thực hành Linux, `systemd`, REST API, PostgreSQL, IAM, Security Group và CloudWatch trong cùng một dự án.
- **Khả năng mở rộng:** ALB và ASG cho phép scale backend từ 2 đến 4 instance; số lượng thiết bị lớn hơn vẫn cần xác thực, HTTPS cho tuyến thiết bị và có thể cần kiến trúc hướng sự kiện.

## Đặc tả API đã được đối chiếu

Qua đối chiếu `backend/main.py` và `backend/app/api/`, FastAPI cung cấp các route sau:

| Method | Route | Thành phần gọi |
| :--- | :--- | :--- |
| `GET` | `/` | Thông tin dịch vụ cơ bản |
| `GET` | `/api/health` | Health check |
| `POST` | `/api/telemetry` | Telemetry từ YOLO UNO |
| `GET` | `/api/devices/{device_id}/latest` | Màn hình latest của dashboard |
| `GET` | `/api/devices/{device_id}/history` | Màn hình history của dashboard |
| `POST` | `/api/devices/{device_id}/commands` | Lệnh điều khiển từ dashboard |
| `GET` | `/api/devices/{device_id}/commands/latest` | Thiết bị thăm dò lệnh |
| `POST` | `/api/devices/{device_id}/commands/{command_id}/ack` | ACK từ thiết bị |

Firmware hỗ trợ `MODE_AUTO`, `MODE_MANUAL`, `FAN_ON`, `FAN_OFF`, `LIGHT_ON`, `LIGHT_OFF`, `CURTAIN_OPEN`, `CURTAIN_CLOSE`. Backend hiện chấp nhận mọi chuỗi lệnh vì `DeviceCommand` chưa có bộ kiểm tra enum đang hoạt động; lệnh không được hỗ trợ sẽ giữ trạng thái `Pending` do firmware từ chối và không gửi ACK. Không dùng route số ít `/api/device/...`.

## Luồng dữ liệu

1. **Telemetry:** YOLO UNO gửi các trường camelCase đến DNS ALB → target group chọn FastAPI instance healthy → Pydantic ánh xạ sang snake_case → SQLAlchemy ghi `telemetry_logs` trên RDS → latest/history qua CloudFront `/api/*` → dashboard.
2. **Lệnh:** dashboard qua CloudFront `/api/*` → ALB → backend ghi `commands.state = "Pending"` → YOLO UNO poll qua ALB → route `commands/latest` trả lệnh chờ cũ nhất trước theo FIFO → phần cứng thực thi.
3. **ACK:** thiết bị gửi ID lệnh → backend chuyển lệnh đó sang `Executed` → telemetry tiếp theo phản ánh trạng thái thiết bị chấp hành. Dịch vụ ACK hiện chỉ tìm theo ID lệnh; kiểm tra lệnh có thuộc đúng thiết bị hay không vẫn là điểm cần gia cố.
4. **Giám sát:** metric mặc định của EC2 cùng metric/log do agent thu thập được gửi tới CloudWatch; RDS cung cấp metric dịch vụ; alarm đánh giá các ngưỡng đã cấu hình.

Do thiết bị thăm dò lệnh thường xuyên, trạng thái `Pending` có thể chỉ xuất hiện trong thời gian rất ngắn. Bằng chứng nên gồm phản hồi khi tạo lệnh và trạng thái `Executed` sau đó trong cơ sở dữ liệu hoặc API.

Các mô hình cơ sở dữ liệu định nghĩa ba bảng `devices`, `telemetry_logs` và `commands`. Các trường telemetry gồm `temperature`, `humidity`, `light_intensity`, `fan_status`, `light_status`, `curtain_status` và `timestamp`.

## Bảo mật và IAM

### Bảng bảo mật mạng

| Nguồn | Đích | Port | Rule |
| :--- | :--- | :---: | :--- |
| Internet / traffic origin CloudFront | ALB Security Group | 80 | Điểm vào API public của deployment đã xác minh |
| ALB Security Group | Backend Security Group | 8000 | Chỉ traffic đến FastAPI target |
| `<ADMIN_IP>/32` | Backend Security Group | 22 | Quản trị có giới hạn khi cần SSH |
| Backend Security Group | RDS Security Group | 5432 | Chỉ PostgreSQL |
| EC2/RDS | CloudWatch | HTTPS | Luồng outbound giám sát |

RDS nằm trong mạng riêng, không public ra Internet. Secret chỉ nằm trong file cục bộ đã loại khỏi Git; EC2 dùng IAM Role thay vì hard-code AWS key. CloudFront cung cấp viewer HTTPS, ALB/ASG cung cấp backend availability và RDS chạy Multi-AZ. Minh chứng hiện tại chưa xác nhận ALB HTTPS, application authentication, rate limiting hoặc WAF Block mode.

### Bảng bảo mật và IAM

| Kiểm soát | Cách triển khai hiện tại | Bằng chứng cần giữ | Hạn chế / bước gia cố tiếp theo |
| :--- | :--- | :--- | :--- |
| Truy cập quản trị | SSH cổng 22 giới hạn theo `<ADMIN_IP>/32` | Quy tắc EC2 Security Group | Rà soát người giữ khóa, thay khóa khi cần; cân nhắc hình thức truy cập được quản lý |
| Cô lập cơ sở dữ liệu | RDS private; cổng 5432 chỉ nhận EC2 Security Group | Cấu hình RDS, subnet group và tham chiếu SG | Rà soát NACL/route và xác minh TLS |
| Thông tin xác thực AWS | EC2 IAM Role có `CloudWatchAgentServerPolicy` phục vụ giám sát | Instance profile và policy đã gắn | Khi phù hợp, thay policy quản lý rộng bằng policy giới hạn tài nguyên đã được rà soát |
| Bí mật ứng dụng | `.env` và `hardware/include/secrets.h` chỉ lưu cục bộ, đã loại khỏi Git | Vị trí file đã che và kết quả `git status` | Chuyển bí mật dùng trong thực tế sang dịch vụ quản lý bí mật đã được phê duyệt |
| API công khai | CloudFront `/api/*` chuyển đến ALB HTTP 80; ALB chuyển tiếp đến backend port 8000 | CloudFront behavior, ALB listener, target health và health request | Bổ sung ALB TLS/custom domain, xác thực, phân quyền và rate limiting trước production |
| Danh tính cơ sở dữ liệu | Người dùng PostgreSQL riêng trong `DATABASE_URL` | Cấu hình kết nối đã che | Giới hạn quyền, thay mật khẩu định kỳ và kiểm toán truy cập |

### Nguyên tắc đặc quyền tối thiểu (Principle of Least Privilege)

Chỉ cấp các hành động cần thiết cho từng danh tính, giới hạn nguồn/đích mạng và tránh dùng thông tin xác thực dài hạn. Workshop không yêu cầu `AdministratorAccess`: người vận hành chỉ cần quyền cấp phát và kiểm tra đã được duyệt; EC2 chỉ cần quyền gửi dữ liệu giám sát; RDS chỉ nhận kết nối PostgreSQL từ EC2 Security Group; người dùng cơ sở dữ liệu chỉ có các quyền ứng dụng thực sự cần. Mọi quyền rộng được cấp tạm để xử lý sự cố phải được ghi lại, giới hạn thời gian, rà soát và gỡ bỏ.

## Mô hình vận hành hiện tại

- CloudFront phục vụ frontend React/Vite từ S3 private và định tuyến `/api/*` của browser đến ALB.
- ASG duy trì hai FastAPI/Uvicorn instance Amazon Linux tại `ap-southeast-1a` và `ap-southeast-1c`, scaling limits từ 2 đến 4.
- Mỗi backend instance dùng EBS gp3 mã hóa và dịch vụ systemd `aws-iot-backend`.
- RDS PostgreSQL Multi-AZ private lưu thiết bị, telemetry và command state; standby phục vụ failover, không phải reader.
- YOLO UNO gọi DNS ALB trực tiếp bằng HTTP polling và ACK định kỳ.
- CloudWatch Agent cùng native metric cung cấp backend logs, operations dashboard, ALB/ASG metrics và tám alarm; alarm đã xác minh hiện chưa có action.

## Lựa chọn mở rộng tương lai và hạn chế hiện tại

Cấu trúc API route theo `device_id` và lược đồ quan hệ có thể hỗ trợ thêm phòng, nhưng phạm vi nghiệm thu hiện tại chỉ là `room_01`. ALB/ASG đã loại bỏ single compute endpoint; cơ chế device polling vẫn tạo độ trễ và tuyến thiết bị trực tiếp chưa có TLS/xác thực được xác minh.

Trong tương lai, nhóm có thể cân nhắc ALB HTTPS với custom domain, xác thực/phân quyền theo thiết bị, MQTT qua AWS IoT Core, hàng đợi như SQS, cache, read replica, container và Infrastructure as Code. Mỗi lựa chọn đều cần thiết kế, đánh giá chi phí/bảo mật, triển khai và kiểm thử riêng.

**Amazon SQS và kiến trúc hướng sự kiện chưa được triển khai trong dự án hiện tại.** Auto Scaling đã được triển khai cho backend.

## Kết quả mong đợi và xử lý sự cố

Mỗi mũi tên trong sơ đồ kiến trúc phải tương ứng với một lời gọi API, quy tắc mạng, thao tác cơ sở dữ liệu hoặc đường đi của metric/log. Nếu chưa rõ một kết nối, hãy xác định nguồn, đích, cổng, danh tính và bằng chứng cần thu thập trước khi cấp phát tài nguyên.

Tiếp theo: [xây dựng hạ tầng AWS](../5.4-AWS-Infrastructure-Setup/).
