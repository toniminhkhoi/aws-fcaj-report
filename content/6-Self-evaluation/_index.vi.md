---
title: "Tự đánh giá"
date: "2026-06-15"
weight: 6
chapter: false
pre: " <b> 6. </b> "
---

Trong thời gian thực hiện dự án từ **01/06/2026** đến **31/07/2026**, em có cơ hội vận dụng kiến thức đã học để xây dựng nguyên mẫu **AWS IoT Monitoring and Control Dashboard**. Em phụ trách chính phần hạ tầng AWS, giám sát hệ thống, tích hợp phần cứng YOLO UNO và hoàn thiện tài liệu kỹ thuật.

### Tóm tắt đóng góp

- Em tham gia nâng cấp hệ thống từ mô hình EC2 và RDS ban đầu lên kiến trúc có CloudFront, S3 private, WAF ở chế độ giám sát, ALB, Auto Scaling Group, RDS PostgreSQL Multi-AZ và CloudWatch.
- Em cấu hình và kiểm tra mạng, security group, IAM role, health check, metric, alarm, log và chính sách sao lưu; đồng thời thu thập ảnh chụp để làm minh chứng.
- Em tích hợp YOLO UNO/ESP32-S3 với các cảm biến và thiết bị chấp hành để gửi telemetry, lấy lệnh điều khiển và xác nhận lệnh đã thực thi qua HTTP REST.
- Em phối hợp với các thành viên phụ trách backend và frontend để kiểm tra toàn bộ luồng dữ liệu và điều khiển.
- Em bổ sung kết quả kiểm thử, giới hạn hiện tại và hướng dẫn bàn giao vào workshop.

### Bảng tự đánh giá

| STT | Tiêu chí | Mức đánh giá | Minh chứng từ dự án |
| :-- | :-- | :-- | :-- |
| 1 | **Kiến thức và kỹ năng chuyên môn** | Tốt | Em đã áp dụng được kiến thức về AWS, mạng, giám sát, cơ sở dữ liệu và hệ thống nhúng vào nguyên mẫu. |
| 2 | **Khả năng học hỏi** | Tốt | Em chủ động đọc tài liệu và thử nghiệm các dịch vụ AWS chưa từng sử dụng trước đó. |
| 3 | **Tính chủ động** | Tốt | Khi phát hiện hạn chế, em tự tìm phương án, kiểm tra lại và đề xuất cách cải thiện kiến trúc. |
| 4 | **Tinh thần trách nhiệm** | Tốt | Em theo sát các nhiệm vụ hạ tầng và phần cứng từ lúc triển khai đến khi kiểm tra và viết tài liệu. |
| 5 | **Kỷ luật và lập kế hoạch** | Khá | Em hoàn thành phạm vi chính, nhưng một số thay đổi được thực hiện gần hạn nên phải chỉnh sửa tài liệu nhiều lần. |
| 6 | **Tiếp nhận góp ý** | Tốt | Em nghiêm túc tiếp thu góp ý và điều chỉnh kiến trúc, workshop cũng như cách trình bày minh chứng. |
| 7 | **Giao tiếp** | Khá | Em có trao đổi tiến độ với nhóm, nhưng đôi lúc nội dung báo cáo còn dài và chưa làm rõ thứ tự ưu tiên. |
| 8 | **Làm việc nhóm** | Tốt | Em phối hợp với các thành viên để kiểm tra điểm kết nối giữa hạ tầng, backend, frontend và phần cứng. |
| 9 | **Tác phong chuyên nghiệp** | Tốt | Em giữ thái độ hợp tác, tôn trọng phần việc của từng thành viên và có ý thức ghi lại các thay đổi kỹ thuật. |
| 10 | **Tư duy giải quyết vấn đề** | Khá | Em xử lý được các lỗi chính, nhưng cách tìm nguyên nhân đôi lúc còn thử nhiều hướng và chưa ghi chép đầy đủ. |
| 11 | **Đóng góp cho dự án** | Tốt | Em hoàn thành phần hạ tầng AWS, giám sát, tích hợp phần cứng và tham gia hoàn thiện workshop. |
| 12 | **Tổng thể** | Tốt | Em đạt được mục tiêu chính của dự án và nhận ra rõ những kỹ năng cần tiếp tục cải thiện. |

### Bài học chính

- Qua dự án, em hiểu rằng một kiến trúc cloud không chỉ cần chạy được mà còn phải quan tâm đến bảo mật, tính sẵn sàng, giám sát, sao lưu, chi phí và khả năng phục hồi.
- Các kết luận về hệ thống cần đi kèm minh chứng cụ thể như health check, phản hồi API, bản ghi cơ sở dữ liệu, metric, log và ảnh chụp.
- Phần cứng, backend, cơ sở dữ liệu và dịch vụ cloud phải thống nhất định dạng dữ liệu cho telemetry, lệnh điều khiển và trạng thái xác nhận.
- Việc cấu hình Multi-AZ hoặc Auto Scaling mới chỉ là điều kiện ban đầu; muốn đánh giá khả năng phục hồi còn cần kiểm thử lỗi có kiểm soát.

### Điểm cần cải thiện

- Em cần lập kế hoạch thay đổi hạ tầng sớm hơn, dự đoán ảnh hưởng và chuẩn bị phương án quay lui trước khi thao tác.
- Em cần trình bày tiến độ, khó khăn và rủi ro ngắn gọn, rõ ý hơn khi trao đổi với nhóm và mentor.
- Khi xử lý lỗi, em nên dùng checklist và ghi lại nguyên nhân gốc thay vì thử nhiều cách rời rạc.
- Em muốn học thêm Infrastructure as Code, triển khai tự động và kiểm thử tích hợp để hạn chế sai lệch do cấu hình thủ công.
- Em cần tìm hiểu sâu hơn về HTTPS đến origin, xác thực người dùng, IAM tối thiểu quyền, WAF ở chế độ chặn và quản lý secret.
- Khi có điều kiện phù hợp, em muốn thực hiện kiểm thử Auto Scaling và RDS failover có kiểm soát để đánh giá hệ thống đầy đủ hơn.
