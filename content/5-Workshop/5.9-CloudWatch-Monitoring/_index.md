---
title: "Monitoring with CloudWatch"
date: "2026-07-28"
weight: 9
chapter: false
pre: " <b> 5.9. </b> "
---

## Overview and objectives

Use native ALB, ASG, EC2, and RDS metrics plus CloudWatch Agent guest metrics and backend logs. The EC2 IAM Role authorizes publishing; the CloudWatch Agent is separate software installed in the backend AMI and running on each ASG instance.

## Monitoring inventory

| Source | Metric/log | Collection path |
| :--- | :--- | :--- |
| ALB | `UnHealthyHostCount`, `HTTPCode_Target_5XX_Count` | Native Application Load Balancer metrics |
| ASG | `GroupInServiceInstances` | Native Auto Scaling group metric |
| EC2 | `CPUUtilization` | Default EC2 metric |
| EC2 guest OS | `mem_used_percent` | CloudWatch Agent configuration; Figure 19 does not prove a memory datapoint |
| EC2 guest OS | `disk_used_percent` | CloudWatch Agent |
| EC2 guest OS | `cpu_usage_idle`, `cpu_usage_user`, `cpu_usage_system` | CloudWatch Agent |
| FastAPI | Backend application log | CloudWatch Agent log collection |
| RDS | `CPUUtilization` | Default RDS metric |
| RDS | `DatabaseConnections` | Default RDS metric |

## Step 1 - Verify role and install the agent

In the EC2 Console, confirm the instance has the IAM instance profile with the approved `CloudWatchAgentServerPolicy`. On EC2 Linux Bash, install the `amazon-cloudwatch-agent` package using the official procedure for the selected distribution, then verify:

```bash
sudo systemctl status amazon-cloudwatch-agent --no-pager
ls -l /opt/aws/amazon-cloudwatch-agent/bin/
```

Do not store AWS access keys in the agent configuration.

## Step 2 - Configure metrics and backend logs

Create `/opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json`:

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

If the active service logs only to journald, either configure the source-defined file logger or use an approved journald collection method; do not point the agent to a nonexistent file.

## Step 3 - Start and enable the agent

In EC2 Linux Bash:

```bash
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl \
  -a fetch-config \
  -m ec2 \
  -c file:/opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json \
  -s
sudo systemctl enable amazon-cloudwatch-agent
sudo systemctl status amazon-cloudwatch-agent --no-pager
```

Check agent diagnostics:

```bash
sudo tail -n 100 /opt/aws/amazon-cloudwatch-agent/logs/amazon-cloudwatch-agent.log
```

## Step 4 - Generate and inspect evidence

1. Call `/api/health` through the ALB and submit one valid telemetry request.
2. Open CloudWatch in the same region.
3. Check log groups `/aws/ec2/aws-iot-dashboard/backend` and `/aws/ec2/aws-iot-dashboard/backend-error`.
4. Open **Metrics → IoTDashboard/EC2** for guest memory/disk/CPU.
5. Open **Metrics → EC2** for `CPUUtilization`.
6. Open **Metrics → RDS** for `CPUUtilization` and `DatabaseConnections`.
7. Open **Metrics → ApplicationELB/Auto Scaling** for target health, 5XX, and in-service instance count.
8. Set an appropriate time range and confirm recent timestamps.

### Backend logs

The captured log stream `/aws/ec2/aws-iot-dashboard/backend` contains recent FastAPI access events. Requests to `/api/health` and `/` returned HTTP `200 OK`, which proves that the backend was reachable at the recorded timestamps. This screenshot does not by itself prove every CloudWatch Agent configuration item.

![FastAPI backend access logs in Amazon CloudWatch Logs](/images/5-Workshop/5.9-cloudwatch/backend-cloudwatch-logs.png)

*Figure 18. FastAPI backend access logs from EC2 displayed in Amazon CloudWatch Logs, including timestamps, requested endpoints, and HTTP status codes.*

### Operations metrics for the current architecture

The current operations dashboard contains eight widgets: EC2 CPU, disk, and memory for both backend instances; ASG in-service capacity; RDS CPU and database connections; ALB unhealthy hosts; and ALB target 5XX errors. This evidence matches the deployed ALB/ASG architecture.

![CloudWatch operations dashboard for ALB, ASG, EC2, and RDS](/images/5-Workshop/5.9-cloudwatch/operations-dashboard.png)
*Figure 19. The eight-widget operations dashboard shows two EC2 series, ASG in-service capacity of 2, no unhealthy ALB targets during the selected range, RDS metrics, and the ALB target 5XX widget.*

![ALB and ASG operational metrics](/images/5-Workshop/5.9-cloudwatch/alb-asg-metrics.png)
*Figure 19a. CloudWatch graph configuration for ALB unhealthy hosts, ALB target 5XX errors, and ASG in-service instances.*

## Step 5 - Create and validate alarms

The CloudWatch console confirms these eight alarm configurations:

| Alarm name | Metric | Condition |
| :--- | :--- | :--- |
| `iot-dashboard-rds-high-connections` | `DatabaseConnections` | ≥10 for one datapoint within 5 minutes |
| `iot-dashboard-rds-high-cpu` | `CPUUtilization` | ≥70% for one datapoint within 5 minutes |
| `iot-dashboard-ec2-high-cpu` | `CPUUtilization` | ≥70% for one datapoint within 5 minutes |
| `ASG-GroupInServiceInstances-Low` | `GroupInServiceInstances` | Fewer than 2 in-service instances during the evaluation window |
| `ALB-HTTPCode-ELB-5XX` | ALB 5XX count | At least one 5XX datapoint during the evaluation window |
| `ALB-UnHealthyHostCount` | `UnHealthyHostCount` | Greater than 0 during the evaluation window |
| `iot-dashboard-ec2-high-disk` | `disk_used_percent` | ≥80% for one datapoint within 5 minutes |
| `iot-dashboard-ec2-high-memory` | `mem_used_percent` | ≥80% for one datapoint within 5 minutes |

Verify the deployed threshold, period, evaluation count, missing-data behavior, and actions instead of assuming that the runbook was applied. The root source README explicitly says SNS is not used; the backend README mentions SNS only as an optional extension, so do not claim an SNS topic/subscription is deployed.

![Eight CloudWatch Alarms monitoring ALB, ASG, EC2, and RDS](/images/5-Workshop/5.9-cloudwatch/cloudwatch-alarms.png)

*Figure 20. Eight CloudWatch Alarms monitor ALB, ASG, EC2, and RDS. The OK and Insufficient data states reflect the available metric data at the time of capture.*

### Alarm state interpretation

- **OK:** the metric has not crossed the configured threshold.
- **In alarm:** the metric crossed the threshold during the evaluation period.
- **Insufficient data:** CloudWatch did not receive enough matching datapoints to evaluate the alarm at that time.

The disk and memory alarms in the screenshot are in the `Insufficient data` state. This does not necessarily indicate a configuration failure; it means that insufficient data was available for the selected metric, dimensions, and evaluation period.

The `Actions` column displays `No actions`, meaning that no notification action is currently attached. The current version uses alarms to evaluate metric states only. Amazon SNS email or notification integration is a future improvement.

## Expected Result

CloudWatch shows recent backend access events, the EC2/RDS and ALB/ASG operational widgets, and eight named alarm configurations with explainable states. The evidence proves what is visible without claiming an SNS notification or a second log group that is not shown.

## Troubleshooting

| Symptom | Check |
| :--- | :--- |
| Agent inactive | Configuration JSON syntax, service log, package installation |
| Access denied | Attached instance profile and policy; avoid local AWS keys |
| No memory/disk metric | `IoTDashboard/EC2` namespace, dimensions, interval, config reload |
| No backend log | Actual log path, read permission, new request, stream timestamp |
| Alarm insufficient data | Metric/dimension/region mismatch or no recent datapoints |
| ALB/ASG alarm is unexpected | Target group dimensions, load balancer dimensions, ASG metric collection, and selected time range |
| RDS metric missing | Correct DB identifier, region, and graph time range |

This project does not use AI Operations, GenAI Observability, Application Signals, resource discovery, or observability pipelines.

Next: [review cost, security, and clean-up](../5.10-Cost-Security-Cleanup/).
