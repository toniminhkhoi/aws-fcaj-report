---
title: "Chi phí, bảo mật và dọn dẹp"
date: "2026-07-28"
weight: 10
chapter: false
pre: " <b> 5.10. </b> "
---

## Tổng quan và mục tiêu

Xác định các yếu tố phát sinh chi phí, rà soát ranh giới bảo mật của mô hình thử nghiệm, lưu lại bằng chứng cần thiết và xóa tài nguyên theo đúng thứ tự phụ thuộc. Không nêu mức giá cố định vì chi phí phụ thuộc vào khu vực, kích thước tài nguyên, thời gian lưu trữ, lưu lượng truyền dữ liệu và bảng giá tại thời điểm sử dụng.

## Bước 1 - Rà soát các yếu tố tạo chi phí

| Tài nguyên | Yếu tố phát sinh chi phí | Cách kiểm soát trong Workshop |
| :--- | :--- | :--- |
| Amazon S3 và CloudFront | Object lưu trữ, request, truyền dữ liệu và gói distribution | Chỉ giữ artifact build; invalidation có chủ đích; rà giới hạn gói |
| AWS WAF | Web ACL/rule và logging | Ba managed rule group đang ở Count/Monitor; rà soát trước khi bật tính năng trả phí |
| Application Load Balancer | Số giờ hoạt động và capacity unit | Xóa sau Workshop nếu không còn sử dụng |
| Amazon EC2 / Auto Scaling | Loại instance, desired capacity và số giờ chạy | ASG đang duy trì hai `t3.micro`; chỉ scale down/xóa sau phê duyệt |
| Amazon EBS | Loại ổ đĩa, dung lượng cấp phát và snapshot | Chọn dung lượng phù hợp; xóa volume/snapshot không còn gắn |
| Amazon RDS for PostgreSQL | DB Multi-AZ, thời gian chạy, lưu trữ và sao lưu | Giữ snapshot cần thiết, sau đó xóa khi đã thu đủ bằng chứng và được duyệt |
| Amazon CloudWatch | Metric tùy chỉnh, lượng log thu nhận/lưu trữ và alarm | Chỉ thu metric hệ điều hành mỗi 60 giây khi cần; đặt thời gian lưu ngắn |
| Truyền dữ liệu | CloudFront, ALB, telemetry thiết bị và lưu lượng liên AZ | Chọn chu kỳ telemetry/polling hợp lý và chỉ cache nội dung tĩnh |

Dùng AWS Pricing Calculator hoặc hóa đơn thực tế để lập bản ước tính có ghi rõ ngày. Không đưa mức giá chưa được kiểm chứng vào báo cáo.

## Bước 2 - Rà soát ranh giới bảo mật

- Dùng danh tính theo nguyên tắc đặc quyền tối thiểu và bật MFA.
- Dùng EC2 IAM Role; không ghi cố định AWS access key trong mã nguồn.
- Giới hạn SSH port 22 theo `<ADMIN_IP>/32`.
- Giữ S3 frontend private với Block Public Access và CloudFront OAC.
- Dùng HTTPS phía viewer qua CloudFront; giữ ba WAF managed rule group ở Count/Monitor cho đến khi kiểm thử chặn.
- Chỉ mở HTTP công khai ở ALB listener; backend port 8000 chỉ nhận từ ALB Security Group.
- Chỉ cho RDS 5432 nhận từ EC2 Security Group.
- Giữ RDS private.
- Loại `.env`, `.pem`, `.key` và `hardware/include/secrets.h` khỏi Git.
- Dùng placeholder trong tài liệu và che thông tin nhạy cảm trên ảnh.
- Xem ALB HTTP chưa có origin TLS và API chưa xác thực là các hạn chế hiện tại.
- Rà soát quy tắc đi ra, thời gian lưu log, người dùng cơ sở dữ liệu và tag tài nguyên.
- Thay mới ngay mọi thông tin bí mật nếu chúng xuất hiện trong Git, lịch sử terminal, ảnh hoặc video demo.

Các kiểm soát hiện có gồm CloudFront viewer HTTPS, WAF monitoring, S3 private/OAC, chuỗi Security Group, EBS mã hóa cho instance ASG, RDS Multi-AZ, backup 7 ngày và manual snapshot. Cải tiến production còn gồm ALB HTTPS/origin TLS, xác thực, phân quyền, quản lý bí mật, WAF Block đã kiểm thử, action thông báo và thiết kế mạng được rà soát.

## Bước 3 - Lưu bằng chứng trước khi dọn dẹp

Trước khi xóa, lưu:

1. sơ đồ kiến trúc và danh mục tài nguyên;
2. trạng thái EC2/dịch vụ và mã commit đã triển khai;
3. bằng chứng bảng/câu truy vấn RDS không chứa thông tin xác thực;
4. kết quả kiểm thử telemetry, lệnh và ACK;
5. ảnh CloudWatch log/metric/alarm; và
6. bằng chứng demo phần cứng/frontend.

Xác nhận người sở hữu snapshot và thời gian phải lưu bằng chứng.

## Danh sách tài nguyên AWS trước khi dọn dẹp

Bảng dưới đây tổng hợp các tài nguyên dự án đang sử dụng. Trước khi dừng hoặc xóa bất kỳ tài nguyên nào, nhóm cần lưu đầy đủ ảnh chụp màn hình, log, kết quả kiểm thử, mã nguồn và video minh họa.

| Tài nguyên | Tên hoặc vai trò | Trạng thái hiện tại | Bằng chứng | Hành động dọn dẹp |
| :--- | :--- | :--- | :--- | :--- |
| Amazon S3 | Bucket frontend private | Block Public Access bật; object được đọc qua CloudFront OAC | Bằng chứng S3 policy/OAC | Chỉ làm rỗng và xóa sau khi CloudFront origin/distribution không còn cần |
| Amazon CloudFront | `iot-dashboard-frontend` | Active; default behavior tới S3 và `/api/*` tới ALB | Bằng chứng Origins/Behaviors | Disable, chờ deploy xong rồi xóa sau khi gỡ phụ thuộc |
| AWS WAF | Web ACL gắn CloudFront | Ba managed rule group ở Count/Monitor | Bằng chứng WAF rules | Chỉ disassociate/xóa theo quyết định dọn CloudFront |
| Application Load Balancer | `iot-backend-alb` | Active, listener HTTP:80 | ALB overview/listener | Xóa sau khi CloudFront không còn dùng ALB origin và ASG đã detach |
| Target Group | `iot-backend-tg` | Hai target Healthy trên HTTP:8000 | Bằng chứng target group | Detach khỏi ALB/ASG rồi xóa |
| Auto Scaling Group | `iot-backend-asg` | Desired 2, giới hạn 2–4; hai instance Healthy/InService | Bằng chứng ASG capacity | Chỉ đổi desired/min trong quá trình shutdown đã duyệt, sau đó xóa ASG |
| Launch Template và AMI | `iot-backend-template`, `iot-backend-ami-v1` | Launch template version 1 và private AMI Available | Bằng chứng Launch Template/AMI | Xóa template và deregister AMI khi không còn ASG/instance phụ thuộc; rà snapshot của AMI |
| Amazon EC2 | Hai backend instance do ASG quản lý | Hai `t3.micro` chạy ở hai Availability Zone | Trang ASG/EC2 | Để ASG terminate trong quy trình đã duyệt; không xóa riêng một instance khi desired capacity vẫn giữ nguyên |
| Amazon EBS | Root volume của các instance ASG | `gp3`, 10 GiB, mã hóa bằng `aws/ebs` | Trang EC2 → Volumes | Kiểm tra `Delete on termination`; chỉ xóa volume/snapshot giữ lại thuộc dự án |
| Amazon RDS for PostgreSQL | `iot-dashboard-db` | Available, PostgreSQL, Multi-AZ; primary 1c/standby 1b; backup 7 ngày | Bằng chứng RDS và AWS CLI | Lưu snapshot/backup đã duyệt, sau đó xóa DB instance |
| RDS snapshot | `iot-dashboard-before-ha-20260730` | Manual snapshot Available | RDS Snapshots | Giữ hoặc xóa theo quyết định lưu dữ liệu đã duyệt |
| DB Subnet Group | `rds-ec2-db-subnet-group-1` | Đang được RDS sử dụng | Phần RDS Connectivity | Chỉ xóa sau khi DB instance và các tài nguyên phụ thuộc đã được xóa |
| ALB/EC2 Security Groups | ALB SG, `iot-backend-sg`, `ec2-rds-1` | Đang áp chuỗi ALB → backend → RDS | Bằng chứng chuỗi Security Group | Xóa sau khi ALB, EC2 và ENI liên quan không còn sử dụng |
| RDS Security Group | `rds-ec2-1` | In use; cho phép kết nối từ EC2 Security Group | Phần RDS Security Group rules | Xóa sau khi RDS và các network interface phụ thuộc đã được xóa |
| IAM Role | `iot-dashboard-cloudwatch-role` | Đang được gắn với EC2 | EC2 Security và trang IAM Role | Tháo role khỏi EC2, kiểm tra tài nguyên phụ thuộc, sau đó xóa role |
| IAM Policy | `CloudWatchAgentServerPolicy` | AWS-managed policy đang gắn với IAM Role | IAM Permissions | Không xóa AWS-managed policy; chỉ tháo hoặc xóa IAM Role của dự án |
| CloudWatch Log Group | `/aws/ec2/aws-iot-dashboard/backend` | Active, chứa FastAPI access logs và HTTP status | CloudWatch Logs | Lưu log cần thiết; sau đó cấu hình retention hoặc xóa log group |
| CloudWatch Dashboard | `ec2-rds-metrics` | Active; hiển thị metric ALB, ASG, EC2 và RDS | CloudWatch Dashboard | Lưu ảnh metric, sau đó xóa dashboard khi không còn cần |
| CloudWatch Alarms | 8 alarm cho ALB, ASG, EC2 và RDS | Một số `OK`, một số `Insufficient data`; chưa gắn notification action | CloudWatch Alarms | Lưu bằng chứng cấu hình, sau đó xóa cả tám alarm |
| Dịch vụ FastAPI backend | `aws-iot-backend` trên các instance ASG | Đang cung cấp REST API và ghi log truy cập | EC2 và CloudWatch Logs | Sao lưu source/service/environment template trước khi xóa ASG/AMI |
| Tệp firmware | `firmware.bin` | Được build thành công bằng PlatformIO trong môi trường `yolo_uno` | Terminal hiển thị `SUCCESS` | Lưu trên máy cục bộ hoặc trong kho artifact; đây không phải tài nguyên AWS cần xóa |

> **Lưu ý:** Không dọn dẹp tài nguyên trước khi lưu đầy đủ ảnh chụp màn hình, log, kết quả kiểm thử, mã nguồn và video minh họa cần thiết. Các volume của instance ASG trong bằng chứng hiện tại đã được mã hóa bằng khóa AWS managed `aws/ebs`.

<!-- TODO: Capture this rendered table as aws-resource-inventory.png -->

## Bước 4 - Chỉ dọn dẹp tài nguyên thuộc dự án

1. Lưu ảnh chụp màn hình, log, mã nguồn, kết quả kiểm thử và video minh họa; dừng telemetry/lệnh mới và đưa thiết bị chấp hành về trạng thái an toàn.
2. Sao lưu source frontend/backend, service file, environment template, metadata AMI/Launch Template và firmware artifact.
3. Quyết định có cần giữ dữ liệu bằng snapshot RDS cuối cùng đã được phê duyệt hay không; tạo snapshot nếu cần.
4. Disable rồi xóa CloudFront distribution sau khi trạng thái deploy hoàn tất; disassociate/xóa WAF web ACL của dự án theo cùng quyết định.
5. Chỉ làm rỗng và xóa S3 frontend private sau khi CloudFront không còn phụ thuộc.
6. Xóa ALB listener/load balancer, detach rồi xóa target group sau khi CloudFront không còn dùng ALB origin.
7. Xóa `iot-backend-asg`; xác nhận các EC2 instance và root volume mã hóa được terminate đúng.
8. Xóa Launch Template, deregister AMI backend và chỉ xóa AMI snapshot thuộc dự án khi không còn cần.
9. Xóa `iot-dashboard-db` khi không còn cần cơ sở dữ liệu và dữ liệu phải lưu; giữ hoặc xóa manual/automated snapshot theo quyết định retention.
10. Xóa cả tám CloudWatch Alarm, dashboard `ec2-rds-metrics`, rồi cấu hình retention hoặc xóa backend log group sau khi lưu bằng chứng.
11. Tháo và xóa `iot-dashboard-cloudwatch-role` cùng instance profile chỉ khi không còn workload nào khác sử dụng. Không xóa AWS-managed policy `CloudWatchAgentServerPolicy`.
12. Chỉ xóa ALB/backend/RDS Security Group sau khi không còn ALB, EC2, RDS, ENI hoặc Security Group phụ thuộc.
13. Chỉ xóa `rds-ec2-db-subnet-group-1` khi RDS không còn sử dụng. Không xóa tài nguyên VPC được chọn sẵn hoặc dùng chung.
14. Kiểm tra lại CloudFront, WAF, S3, ALB, target group, ASG, EC2, AMI snapshot, EBS, RDS, CloudWatch, Billing/Cost Explorer và tài nguyên theo tag.

Repository không chứa stack/state CloudFormation, SAM, CDK hoặc Terraform cho deployment này, vì vậy phải kiểm kê và xóa thủ công theo thứ tự phụ thuộc thay vì xóa một stack.

Dừng RDS chỉ là biện pháp tạm thời và chịu giới hạn của dịch vụ; RDS có thể tự khởi động lại. Muốn tránh chi phí dài hạn, cần xóa cơ sở dữ liệu và các tài nguyên tính phí khác theo quyết định lưu giữ của nhóm.

## Bước 5 - Xác minh kết quả dọn dẹp

- Kiểm kê lại tài nguyên theo tag trong đúng khu vực.
- Kiểm tra CloudFront/WAF còn hoạt động, S3 bucket chưa rỗng, ALB/target group, instance ASG, AMI snapshot, volume EBS không gắn, snapshot RDS, log group và alarm không còn sử dụng.
- Nếu không xóa được Security Group, hãy tìm ENI hoặc tài nguyên phụ thuộc thay vì cưỡng chế xóa.
- Nếu không xóa được cơ sở dữ liệu, rà soát chế độ bảo vệ xóa và yêu cầu snapshot với người sở hữu.
- Ghi ID của từng tài nguyên đã xóa trong bằng chứng dọn dẹp; không để lộ thông tin xác thực.

## Kết quả mong đợi

Các bằng chứng cần thiết được lưu lại; không còn tài nguyên có tính phí của dự án nằm ngoài phạm vi được phê duyệt; tài nguyên dùng chung không bị ảnh hưởng; phần rà soát bảo mật phản ánh đúng các hạn chế hiện tại và không tuyên bố hệ thống đã sẵn sàng để vận hành thực tế.

## Xử lý sự cố

| Hiện tượng | Nội dung cần kiểm tra |
| :--- | :--- |
| Không xóa được Security Group | Tìm ENI, EC2, RDS hoặc Security Group tham chiếu còn phụ thuộc |
| Không xóa được VPC/subnet | Kiểm tra Internet Gateway, route table association, DB Subnet Group và ENI |
| RDS bị chặn khi xóa | Chế độ bảo vệ xóa, tên snapshot cuối, bản sao lưu tự động được giữ lại và phê duyệt của người sở hữu |
| EBS vẫn phát sinh phí | Kiểm tra volume không gắn và các snapshot thực sự thuộc dự án |
| CloudWatch vẫn phát sinh phí | Thời gian lưu/lượng log thu nhận và alarm; xác nhận Agent đã dừng cùng EC2 |
| Chưa rõ tài nguyên thuộc về ai | Dừng xóa, dùng tag/danh mục tài nguyên và xin người sở hữu xác nhận |

Tiếp theo: [ghi nhận kết quả, thách thức và cải tiến tương lai](../5.11-Results-Challenges-Future/).
