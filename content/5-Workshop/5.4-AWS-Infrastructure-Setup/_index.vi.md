---
title: "Thiết lập hạ tầng AWS"
date: "2026-07-28"
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

## Tổng quan và mục tiêu

Xây dựng nền tảng mạng, bảo mật biên, cân bằng tải, Auto Scaling, lưu trữ và cơ sở dữ liệu cho hệ thống đang vận hành. Trong ghi chú và ảnh chụp, hãy dùng giá trị giữ chỗ; không công khai ID tài khoản, mật khẩu, endpoint riêng tư hoặc khóa.

## Bước 1 - Chọn khu vực và lập kế hoạch địa chỉ

Trong AWS Console, chọn khu vực đã thống nhất cho dự án. Workshop dùng **Asia Pacific (Singapore), `ap-southeast-1`** làm khu vực mặc định. Ghi lại các dải CIDR không chồng lấn cho VPC, một public subnet và ít nhất hai DB subnet thuộc hai Availability Zone.

**Kết quả mong đợi:** mọi tài nguyên bên dưới nằm trong cùng khu vực và dùng tiền tố tên đã thống nhất.

## Bước 2 - Tạo hoặc chọn VPC và subnet

1. Mở **VPC → Your VPCs**, tạo hoặc chọn VPC của dự án.
2. Bật DNS resolution và DNS hostnames.
3. Tạo/chọn hai public application subnet ở `ap-southeast-1a` và `ap-southeast-1c` cho ALB và Auto Scaling Group.
4. Gắn Internet Gateway vào VPC.
5. Thêm `0.0.0.0/0 → Internet Gateway` vào route table của các application subnet.
6. Tạo hoặc chọn các subnet cơ sở dữ liệu private ở ít nhất hai Availability Zone; cấu hình đã kiểm chứng dùng primary tại `ap-southeast-1c` và standby tại `ap-southeast-1b`. Không thêm route tới Internet Gateway cho các DB subnet.
7. Trong **RDS → Subnet groups**, tạo DB Subnet Group chứa cả hai DB subnet.

**Kết quả mong đợi:** ALB trải trên hai application subnet, ASG duy trì hai backend ở hai Availability Zone và các subnet cơ sở dữ liệu vẫn nằm trong mạng riêng.

## Bước 2A - Cấu hình S3 private, CloudFront và WAF

1. Build frontend React + Vite và tải các artifact trong `dist` lên bucket S3 của dự án.
2. Bật **Block all public access** cho bucket.
3. Tạo CloudFront Origin Access Control (OAC) và chỉ cho distribution đọc các object S3 cần thiết.
4. Cấu hình hai CloudFront origin: bucket S3 private cho static file và `iot-backend-alb` cho API.
5. Giữ default behavior `*` trỏ tới S3 origin với managed optimized caching.
6. Tạo behavior `/api/*` có độ ưu tiên cao hơn cho ALB origin, tắt cache, chuyển tiếp các viewer value cần thiết ngoại trừ header `Host` và cho phép các HTTP method mà API sử dụng.
7. Gắn web ACL do CloudFront tạo. Ba managed rule group hiện chạy ở **Count/Monitor mode**, nên chỉ quan sát request và chưa chặn request.

CloudFront cung cấp HTTPS cho trình duyệt. Trong cấu hình đã kiểm chứng, CloudFront kết nối đến ALB origin qua HTTP và YOLO UNO cũng gọi trực tiếp ALB qua HTTP; không mô tả hai tuyến này là TLS đầu cuối.

![Hai CloudFront origin cho frontend S3 private và ALB API](/images/5-Workshop/5.4-aws-infrastructure/cloudfront-distribution-origins.png)
*Hình 3a. Distribution có hai origin riêng cho S3 private và ALB.*

![Hai CloudFront behavior cho static content và API](/images/5-Workshop/5.4-aws-infrastructure/cloudfront-behaviors.png)
*Hình 3b. Default behavior phục vụ nội dung S3; `/api/*` có ưu tiên cao hơn và dùng ALB origin.*

![Bucket S3 private với Block Public Access và CloudFront OAC](/images/5-Workshop/5.4-aws-infrastructure/s3-private-oac.png)
*Hình 3c. Bucket frontend vẫn private và chỉ cho CloudFront distribution đọc object qua OAC.*

![AWS WAF web ACL có ba managed rule group](/images/5-Workshop/5.4-aws-infrastructure/waf-web-acl-three-rules.png)
*Hình 3d. Ba AWS managed rule group được gắn với distribution và đang ở Count mode.*

## Bước 3 - Tạo Security Group

Tạo các Security Group của ALB, backend và RDS trong cùng VPC. Môi trường đã triển khai dùng ALB Security Group cho lưu lượng HTTP công khai, `iot-backend-sg` cho backend, `ec2-rds-1` cho kết nối từ EC2 tới RDS và `rds-ec2-1` cho quy tắc phía RDS.

| Security Group | Loại | Nguồn | Mục đích |
| :--- | :---: | :--- | :--- |
| ALB Security Group | HTTP 80 | `0.0.0.0/0` | Nhận request từ CloudFront và thiết bị YOLO UNO |
| `iot-backend-sg` | Custom TCP 8000 | ALB Security Group | Chỉ nhận FastAPI traffic từ ALB |
| `iot-backend-sg` | SSH 22 | `<ADMIN_IP>/32` | Quản trị có giới hạn |
| `ec2-rds-1` → `rds-ec2-1` | PostgreSQL 5432 | Tham chiếu EC2 Security Group | Chỉ EC2 tới RDS |

Backend không còn công khai trực tiếp cổng 8000. Trình duyệt gọi `/api/*` qua CloudFront, còn YOLO UNO dùng DNS của ALB theo đúng route đã kiểm chứng. ALB forward tới target group trên cổng 8000. RDS không mở PostgreSQL 5432 cho `0.0.0.0/0`; cơ sở dữ liệu chỉ nhận kết nối từ EC2 Security Group.

Hai ảnh dưới đây tách riêng quy tắc phía EC2 và quan hệ Security Group phía RDS, đồng thời đã che IP quản trị cùng các định danh nhạy cảm.

![Chuỗi Security Group từ ALB tới backend và RDS](/images/5-Workshop/5.4-aws-infrastructure/security-group-chain.png)
*Hình 7a. Chuỗi Security Group giới hạn luồng `Internet/CloudFront → ALB:80 → backend:8000 → RDS:5432`.*

![RDS chỉ cho phép PostgreSQL từ backend Security Group](/images/5-Workshop/5.4-aws-infrastructure/rds-security-group.png)
*Hình 7b. RDS Security Group cho phép TCP 5432 từ Security Group của backend, không mở cơ sở dữ liệu ra Internet.*

## Bước 4 - Tạo EC2 IAM Role

1. Mở **IAM → Roles → Create role**.
2. Chọn trusted entity **AWS service → EC2**.
3. Chỉ gắn `CloudWatchAgentServerPolicy` khi CloudWatch Agent cần gửi metric hoặc log.
4. Đặt tên role là `iot-dashboard-cloudwatch-role`, đúng với môi trường đã triển khai, rồi tạo instance profile.

Không tạo access key dài hạn. Role đang dùng AWS-managed policy `CloudWatchAgentServerPolicy`, cho phép CloudWatch Agent gửi log và metric mà không cần ghi cố định AWS access key. Role chỉ cấp quyền; CloudWatch Agent được cài đặt riêng ở mục 5.9. Trước khi dùng trong production, cần rà soát và thu hẹp quyền thay vì mặc định policy được quản lý đã đáp ứng hoàn toàn nguyên tắc đặc quyền tối thiểu.

Trang Security của EC2 và phần chi tiết IAM Role xác nhận role đã được gắn cùng AWS-managed policy.

![IAM Role và CloudWatchAgentServerPolicy được gắn với EC2](/images/5-Workshop/5.4-aws-infrastructure/ec2-iam-role.png)
*Hình 5. EC2 được gắn IAM Role `iot-dashboard-cloudwatch-role`, và role này sử dụng `CloudWatchAgentServerPolicy` để CloudWatch Agent gửi log và metric mà không cần hard-code AWS access key.*

## Bước 5 - Chuẩn bị AMI, Launch Template, ASG và EBS

1. Xác minh instance nguồn chạy FastAPI ổn định trước khi tạo AMI.
2. Tạo AMI `iot-backend-ami-v1` và dùng AMI này trong Launch Template `iot-backend-template`.
3. Chọn loại instance `t3.micro`, gắn IAM instance profile, key pair và backend Security Group trong Launch Template.
4. Cấu hình ổ đĩa gốc EBS `gp3`, 10 GiB, bật mã hóa bằng khóa AWS managed `aws/ebs`.
5. Tạo Auto Scaling Group `iot-backend-asg` trên hai application subnet với `min/desired/max = 2/2/4`.
6. Chờ hai instance chuyển sang **InService**, **Healthy** và vượt qua status checks.

Launch Template và ASG thay cho việc phụ thuộc vào một EC2 duy nhất. Không ghi lại hoặc đưa địa chỉ IP instance vào frontend hay firmware; điểm vào ổn định của backend là DNS của ALB.

![AMI và Launch Template của backend](/images/5-Workshop/5.4-aws-infrastructure/launch-template-ami.png)
*Hình 4a. Launch Template phiên bản 1 dùng AMI riêng của FastAPI backend.*

![ASG duy trì hai backend ở hai Availability Zone](/images/5-Workshop/5.4-aws-infrastructure/asg-capacity-instances.png)
*Hình 4b. `iot-backend-asg` duy trì desired capacity bằng 2, giới hạn 2–4 và có hai instance Healthy/InService.*

![Các volume EBS của backend được mã hóa](/images/5-Workshop/5.4-aws-infrastructure/ebs-encryption-kms.png)
*Hình 4c. Các volume `gp3` 10 GiB mới của ASG được mã hóa bằng khóa AWS managed `aws/ebs`.*

### Bước 5A - Tạo Target Group và Application Load Balancer

1. Tạo target group `iot-backend-tg`, target type **Instance**, protocol/port `HTTP:8000`.
2. Cấu hình health check tại `/api/health`.
3. Tạo internet-facing Application Load Balancer `iot-backend-alb` trên hai application subnet.
4. Tạo listener `HTTP:80` và forward 100% request tới `iot-backend-tg`.
5. Gắn target group với ASG và xác minh cả hai target đều **Healthy**.

![Application Load Balancer đang Active](/images/5-Workshop/5.4-aws-infrastructure/alb-overview.png)
*Hình 5a. `iot-backend-alb` ở trạng thái Active và trải trên hai Availability Zone.*

![ALB listener forward tới target group](/images/5-Workshop/5.4-aws-infrastructure/alb-listener-forwarding.png)
*Hình 5b. Listener HTTP:80 forward toàn bộ request tới `iot-backend-tg`.*

![Hai target backend Healthy](/images/5-Workshop/5.4-aws-infrastructure/target-group-healthy.png)
*Hình 5c. Target group có hai target Healthy, mỗi target ở một Availability Zone.*

## Bước 6 - Tạo Amazon RDS for PostgreSQL

1. Mở **RDS → Databases → Create database**, chọn PostgreSQL.
2. Chọn loại instance `db.t4g.micro` và cấu hình lưu trữ đã duyệt.
3. Đặt tên cơ sở dữ liệu ban đầu là `iot_dashboard`.
4. Chọn VPC của dự án, DB Subnet Group `rds-ec2-db-subnet-group-1` và các RDS Security Group đã triển khai.
5. Giữ Internet access gateway ở trạng thái Disabled như cấu hình RDS thực tế.
6. Lưu mật khẩu quản trị an toàn dưới dạng `<DB_PASSWORD>`; không đưa mật khẩu thật vào ảnh hoặc Git.
7. Bật Multi-AZ, automated backup với thời gian lưu 7 ngày và tạo manual snapshot trước thay đổi lớn.
8. Chờ `iot-dashboard-db` chuyển sang **Available**; cấu hình đã kiểm chứng có primary tại `ap-southeast-1c` và standby tại `ap-southeast-1b`. Lưu `<RDS_ENDPOINT>` ở nơi riêng tư.

Standby Multi-AZ phục vụ failover và không phải read replica để ứng dụng đọc dữ liệu. Workshop chưa triển khai RDS Proxy, public endpoint hoặc IAM database authentication.

Trang Summary và Connectivity & security của RDS xác nhận PostgreSQL engine, DB class, DB Subnet Group, Availability Zone và trạng thái tắt Internet access gateway.

![Amazon RDS PostgreSQL ở trạng thái Available và sử dụng DB Subnet Group](/images/5-Workshop/5.4-aws-infrastructure/rds-postgresql-available.png)
*Hình 6. Amazon RDS for PostgreSQL `iot-dashboard-db` ở trạng thái Available, sử dụng DB Subnet Group `rds-ec2-db-subnet-group-1` và tắt Internet access gateway.*

![Primary và standby của RDS Multi-AZ](/images/5-Workshop/5.4-aws-infrastructure/rds-primary-standby-az.png)
*Hình 6a. AWS CLI xác nhận Multi-AZ được bật, primary ở `ap-southeast-1c` và standby ở `ap-southeast-1b`.*

![RDS automated backup được lưu 7 ngày](/images/5-Workshop/5.4-aws-infrastructure/rds-backup-retention.png)
*Hình 6b. Automated backup được bật với retention period 7 ngày.*

![Manual snapshot trước thay đổi hạ tầng](/images/5-Workshop/5.4-aws-infrastructure/rds-manual-snapshot.png)
*Hình 6c. Manual snapshot hoàn tất và sẵn sàng phục hồi khi cần.*

## Bước 7 - Xác minh truy cập và mạng

Nếu cần quản trị một instance cụ thể, kết nối từ Windows PowerShell bằng IP quản trị được cấp tạm thời:

```powershell
ssh -i "$env:USERPROFILE\.ssh\<KEY_FILE>.pem" <EC2_USER>@<EC2_PUBLIC_IP>
```

Từ EC2 Linux Bash, kiểm tra DNS và TCP:

```bash
getent hosts <RDS_ENDPOINT>
nc -vz <RDS_ENDPOINT> 5432
```

Nếu chưa có `nc`, hãy cài gói netcat phù hợp với bản phân phối Linux. Kết nối TCP thành công chỉ xác nhận route và Security Group hoạt động; kết quả này chưa chứng minh thông tin đăng nhập cơ sở dữ liệu là chính xác.

Từ máy khách, kiểm tra health endpoint qua ALB:

```powershell
curl.exe -sS -i "http://<ALB_DNS_NAME>/api/health"
```

## Kết quả mong đợi và bằng chứng

Các hình trên cung cấp bằng chứng đã che thông tin nhạy cảm về AMI/Launch Template, ASG hai instance, EBS mã hóa, ALB và hai target Healthy, IAM Role đã gắn, RDS Multi-AZ, backup/snapshot và chuỗi Security Group. Cần lưu riêng kết quả kiểm tra cổng từ backend tới RDS và health check qua ALB.

## Xử lý sự cố

| Hiện tượng | Nội dung cần kiểm tra |
| :--- | :--- |
| SSH hết thời gian chờ | IP công khai, route table, Internet Gateway, `<ADMIN_IP>/32` và tường lửa cục bộ |
| SSH từ chối quyền | Tài khoản đăng nhập và quyền của khóa trên máy cục bộ; không mở rộng SG để xử lý lỗi khóa |
| RDS hết thời gian chờ | Endpoint/khu vực, route của DB subnet, nguồn trong RDS SG và network ACL |
| RDS từ chối kết nối | Thông tin đăng nhập, đúng cổng và cơ sở dữ liệu đang ở trạng thái `Available` |
| EC2 không gửi được metric | IAM instance profile đã gắn và kết nối HTTPS đi ra |
| Trình duyệt gọi API thất bại | CloudFront behavior `/api/*`, ALB listener, target health và backend log |
| YOLO UNO không tới backend | DNS ALB trong firmware, Wi-Fi/DNS, ALB listener và target health |

Tiếp theo: [triển khai FastAPI và kết nối PostgreSQL](../5.5-Backend-and-Database/).
