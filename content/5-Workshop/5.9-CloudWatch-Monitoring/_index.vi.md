---
title: "Giám sát bằng CloudWatch"
date: "2026-07-28"
weight: 9
chapter: false
pre: " <b> 5.9. </b> "
---

## Tổng quan và mục tiêu

Sử dụng metric mặc định của ALB, ASG, EC2 và RDS, cùng metric hệ điều hành và log backend do CloudWatch Agent thu thập. EC2 IAM Role cấp quyền gửi dữ liệu; CloudWatch Agent được cài trong AMI backend và chạy trên từng instance của ASG.

## Danh mục giám sát

| Nguồn | Metric/log | Cách thu thập |
| :--- | :--- | :--- |
| ALB | `UnHealthyHostCount`, `HTTPCode_Target_5XX_Count` | Metric mặc định của Application Load Balancer |
| ASG | `GroupInServiceInstances` | Metric của Auto Scaling Group |
| EC2 | `CPUUtilization` | Metric mặc định EC2 |
| Guest OS EC2 | `mem_used_percent` | Cấu hình CloudWatch Agent; Hình 19 không chứng minh có datapoint bộ nhớ |
| Guest OS EC2 | `disk_used_percent` | CloudWatch Agent |
| Guest OS EC2 | `cpu_usage_idle`, `cpu_usage_user`, `cpu_usage_system` | CloudWatch Agent |
| FastAPI | Log ứng dụng backend | CloudWatch Agent đọc file log |
| RDS | `CPUUtilization` | Metric mặc định RDS |
| RDS | `DatabaseConnections` | Metric mặc định RDS |

## Bước 1 - Xác minh IAM Role và cài Agent

Trong EC2 Console, xác nhận instance đã gắn IAM instance profile có `CloudWatchAgentServerPolicy` được phê duyệt. Trên EC2 Linux Bash, cài gói `amazon-cloudwatch-agent` theo hướng dẫn chính thức cho bản phân phối Linux đã chọn, rồi kiểm tra:

```bash
sudo systemctl status amazon-cloudwatch-agent --no-pager
ls -l /opt/aws/amazon-cloudwatch-agent/bin/
```

Không lưu AWS access key trong cấu hình của Agent.

## Bước 2 - Cấu hình metric và backend log

Tạo `/opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json`:

```json
{
  "agent": {
    "metrics_collection_interval": 60,
    "run_as_user": "root"
  },
  "metrics": {
    "namespace": "IoTDashboard/EC2",
    "append_dimensions": {
      "InstanceId": "${aws:InstanceId}"
    },
    "metrics_collected": {
      "mem": {
        "measurement": ["mem_used_percent"]
      },
      "disk": {
        "measurement": ["used_percent"],
        "resources": ["/"]
      },
      "cpu": {
        "measurement": ["cpu_usage_idle", "cpu_usage_user", "cpu_usage_system"],
        "totalcpu": true
      }
    }
  },
  "logs": {
    "logs_collected": {
      "files": {
        "collect_list": [
          {
            "file_path": "/var/log/aws-iot-backend/backend.log",
            "log_group_name": "/aws/ec2/aws-iot-dashboard/backend",
            "log_stream_name": "{instance_id}/backend",
            "timezone": "UTC"
          },
          {
            "file_path": "/var/log/aws-iot-backend/backend-error.log",
            "log_group_name": "/aws/ec2/aws-iot-dashboard/backend-error",
            "log_stream_name": "{instance_id}/backend-error",
            "timezone": "UTC"
          }
        ]
      }
    }
  }
}
```

Nếu dịch vụ chỉ ghi log vào journald, hãy cấu hình ghi log ra file theo mã nguồn hoặc dùng phương thức thu thập journald đã được phê duyệt; không cấu hình Agent đọc một file không tồn tại.

## Bước 3 - Khởi động Agent và bật chế độ tự chạy

Trong EC2 Linux Bash:

```bash
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl \
  -a fetch-config \
  -m ec2 \
  -c file:/opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json \
  -s
sudo systemctl enable amazon-cloudwatch-agent
sudo systemctl status amazon-cloudwatch-agent --no-pager
```

Xem log chẩn đoán của Agent:

```bash
sudo tail -n 100 /opt/aws/amazon-cloudwatch-agent/logs/amazon-cloudwatch-agent.log
```

## Bước 4 - Tạo và kiểm tra bằng chứng

1. Gọi `/api/health` qua ALB và gửi một yêu cầu telemetry hợp lệ.
2. Mở CloudWatch trong cùng khu vực.
3. Kiểm tra log group `/aws/ec2/aws-iot-dashboard/backend` và `/aws/ec2/aws-iot-dashboard/backend-error`.
4. Mở **Metrics → IoTDashboard/EC2** để xem bộ nhớ, ổ đĩa và CPU của hệ điều hành khách.
5. Mở **Metrics → EC2** cho `CPUUtilization`.
6. Mở **Metrics → RDS** cho `CPUUtilization` và `DatabaseConnections`.
7. Mở **Metrics → ApplicationELB/Auto Scaling** để xem target health, 5XX và số instance InService.
8. Chọn khoảng thời gian phù hợp và xác nhận có dữ liệu với timestamp mới.

### Log backend

Log stream `/aws/ec2/aws-iot-dashboard/backend` trong ảnh chứa các sự kiện truy cập FastAPI gần đây. Yêu cầu tới `/api/health` và `/` đều trả về HTTP `200 OK`, cho thấy backend có thể truy cập được tại các thời điểm đã ghi nhận. Tuy nhiên, riêng ảnh này chưa đủ để xác nhận toàn bộ cấu hình của CloudWatch Agent.

![FastAPI backend access logs trong Amazon CloudWatch Logs](/images/5-Workshop/5.9-cloudwatch/backend-cloudwatch-logs.png)

*Hình 18. Log truy cập của FastAPI backend trên EC2 được thu thập và hiển thị trong Amazon CloudWatch Logs, bao gồm timestamp, endpoint và HTTP status.*

### Metric vận hành của kiến trúc hiện tại

Dashboard vận hành hiện tại có tám widget: CPU, disk và memory của hai backend EC2; số instance InService của ASG; CPU và số kết nối RDS; số ALB target unhealthy; và lỗi ALB target 5XX. Bằng chứng này khớp với kiến trúc ALB/ASG đang triển khai.

![CloudWatch operations dashboard cho ALB, ASG, EC2 và RDS](/images/5-Workshop/5.9-cloudwatch/operations-dashboard.png)
*Hình 19. Dashboard tám widget hiển thị hai chuỗi EC2, ASG có 2 instance InService, không có ALB target unhealthy trong khoảng đã chọn, các metric RDS và widget ALB target 5XX.*

![Metric vận hành ALB và ASG](/images/5-Workshop/5.9-cloudwatch/alb-asg-metrics.png)
*Hình 19a. Cấu hình đồ thị CloudWatch cho ALB unhealthy hosts, ALB target 5XX và ASG in-service instances.*

## Bước 5 - Tạo và xác minh các alarm

Bảng điều khiển CloudWatch xác nhận tám cấu hình alarm sau:

| Tên alarm | Metric | Điều kiện |
| :--- | :--- | :--- |
| `iot-dashboard-rds-high-connections` | `DatabaseConnections` | ≥10 với một datapoint trong 5 phút |
| `iot-dashboard-rds-high-cpu` | `CPUUtilization` | ≥70% với một datapoint trong 5 phút |
| `iot-dashboard-ec2-high-cpu` | `CPUUtilization` | ≥70% với một datapoint trong 5 phút |
| `ASG-GroupInServiceInstances-Low` | `GroupInServiceInstances` | Ít hơn 2 instance InService trong khoảng đánh giá |
| `ALB-HTTPCode-ELB-5XX` | Số lỗi 5XX của ALB | Có ít nhất một datapoint 5XX trong khoảng đánh giá |
| `ALB-UnHealthyHostCount` | `UnHealthyHostCount` | Lớn hơn 0 trong khoảng đánh giá |
| `iot-dashboard-ec2-high-disk` | `disk_used_percent` | ≥80% với một datapoint trong 5 phút |
| `iot-dashboard-ec2-high-memory` | `mem_used_percent` | ≥80% với một datapoint trong 5 phút |

Hãy kiểm tra ngưỡng, chu kỳ, số lần đánh giá, cách xử lý dữ liệu thiếu và action thực tế của từng alarm, thay vì mặc định cấu hình trong tài liệu đã được áp dụng. README gốc cho biết dự án không dùng SNS; README của backend chỉ nêu SNS như một hướng mở rộng, vì vậy không được tuyên bố đã triển khai SNS topic hoặc subscription.

![Tám CloudWatch Alarms giám sát ALB, ASG, EC2 và RDS](/images/5-Workshop/5.9-cloudwatch/cloudwatch-alarms.png)

*Hình 20. Tám CloudWatch Alarms giám sát ALB, ASG, EC2 và RDS. Trạng thái OK hoặc Insufficient data phản ánh dữ liệu metric tại thời điểm chụp.*

### Ý nghĩa trạng thái alarm

- **OK:** metric hiện chưa vượt ngưỡng cấu hình.
- **In alarm:** metric đã vượt ngưỡng trong khoảng đánh giá.
- **Insufficient data:** CloudWatch chưa nhận đủ datapoint để đánh giá alarm tại thời điểm đó.

Hai alarm về ổ đĩa và bộ nhớ đang ở trạng thái `Insufficient data`. Trạng thái này không nhất thiết là lỗi cấu hình; CloudWatch chỉ chưa có đủ dữ liệu khớp với metric, dimension và khoảng đánh giá.

Cột `Actions` hiển thị `No actions`, nghĩa là các alarm chưa được gắn hành động thông báo. Phiên bản hiện tại chỉ dùng alarm để theo dõi trạng thái metric. Việc tích hợp Amazon SNS để gửi email hoặc thông báo là hướng phát triển trong tương lai.

## Kết quả mong đợi

CloudWatch hiển thị các sự kiện truy cập backend gần đây, các widget EC2/RDS và ALB/ASG, cùng tám cấu hình alarm với trạng thái có thể giải thích. Phần bằng chứng chỉ mô tả những gì quan sát được; không khẳng định SNS đã gửi thông báo hoặc log group thứ hai đã xuất hiện.

## Xử lý sự cố

| Hiện tượng | Nội dung cần kiểm tra |
| :--- | :--- |
| Agent không hoạt động | Cú pháp JSON, log dịch vụ và quá trình cài gói |
| Bị từ chối quyền | Instance profile và policy đã gắn; không dùng AWS key cục bộ |
| Không có metric bộ nhớ/ổ đĩa | Namespace `IoTDashboard/EC2`, dimension, chu kỳ thu thập và việc nạp lại cấu hình |
| Không có log backend | Đường dẫn log thực tế, quyền đọc, yêu cầu mới và timestamp của luồng |
| Alarm thiếu dữ liệu | Sai metric/dimension/khu vực hoặc không có điểm dữ liệu mới |
| Alarm ALB/ASG ngoài dự kiến | Kiểm tra dimension của target group/load balancer, việc bật ASG metric và khoảng thời gian |
| Thiếu metric RDS | Đúng DB identifier, khu vực và khoảng thời gian của biểu đồ |

Dự án không sử dụng AI Operations, GenAI Observability, Application Signals, tính năng khám phá tài nguyên hoặc quy trình quan sát nâng cao.

Tiếp theo: [rà soát chi phí, bảo mật và dọn dẹp](../5.10-Cost-Security-Cleanup/).
