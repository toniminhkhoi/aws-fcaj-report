---
title: "Kết quả, thách thức và hướng cải tiến"
date: "2026-07-28"
weight: 11
chapter: false
pre: " <b> 5.11. </b> "
---

## Tổng quan

Phần này tổng hợp các kết quả đã được xác minh, những điều chỉnh riêng của dự án, đóng góp của từng thành viên, bài học rút ra, giới hạn hiện tại và hướng cải tiến tiếp theo. Kết luận được đối chiếu từ mã nguồn, ma trận kiểm thử T01–T12, ảnh chụp AWS, bản ghi PostgreSQL, giao diện dashboard và video demo phần cứng.

## Kết quả đạt được

| Kết quả | Trạng thái | Bằng chứng |
| :--- | :---: | :--- |
| Backend FastAPI hoạt động và phản hồi health check | **Đạt** | Phản hồi health check và trạng thái `systemd` trong [mục 5.5](../5.5-Backend-and-Database/) |
| Gửi telemetry và lưu dữ liệu vào PostgreSQL | **Đạt** | Phản hồi API và bản ghi tương ứng trong RDS tại [mục 5.8](../5.8-End-to-End-Testing/) |
| Hiển thị dữ liệu mới nhất và lịch sử trên dashboard | **Đạt** | Các thẻ telemetry, biểu đồ lịch sử và request HTTP 200 trong mục [5.7](../5.7-Frontend-Integration/) và [5.8](../5.8-End-to-End-Testing/) |
| Tạo lệnh và chuyển trạng thái `Pending` → `Executed` | **Đạt** | ID lệnh, phản hồi ACK và trạng thái PostgreSQL trong mục [5.5](../5.5-Backend-and-Database/) và [5.8](../5.8-End-to-End-Testing/) |
| Điều khiển quạt, đèn và rèm vật lý | **Đạt** | Ảnh đối chiếu dashboard–phần cứng và video demo end-to-end trong [mục 5.8](../5.8-End-to-End-Testing/) |
| Thu thập log, metric và cấu hình alarm trên CloudWatch | **Đạt** | Log backend, bốn widget giám sát và năm cấu hình alarm trong [mục 5.9](../5.9-CloudWatch-Monitoring/) |
| Kiểm thử toàn bộ luồng end-to-end | **Đạt** | Cả 12 test case T01–T12 trong ma trận kiểm thử đều có trạng thái Đạt |

Phạm vi kiểm thử là mô hình Smart Room đã triển khai với `device_id=room_01`. Các kết quả trên chỉ áp dụng cho môi trường đã được trình diễn và không được suy rộng ra ngoài phạm vi này.

## Các điều chỉnh riêng của dự án (Project Customizations)

Dự án không sao chép nguyên trạng một bài hướng dẫn có sẵn. Nhóm đã điều chỉnh và đối chiếu các nội dung sau:

- mô hình `room_01` kết nối telemetry vật lý, lịch sử trên dashboard và trạng thái thiết bị chấp hành;
- vòng đời lệnh FastAPI/PostgreSQL lưu `Pending`, `Executed` và ACK từ thiết bị;
- 8 lệnh firmware cho chế độ tự động/thủ công cùng khả năng điều khiển trực tiếp quạt, đèn và rèm;
- các ngưỡng điều khiển, sơ đồ GPIO, LCD1602, thời gian kết nối lại và cơ chế khôi phục ACK bằng ESP32 Preferences;
- dashboard React/Vite có biểu đồ telemetry, bảng điều khiển thiết bị và các đề xuất vận hành dựa trên luật;
- kết nối RDS riêng thông qua tham chiếu Security Group, không mở cơ sở dữ liệu công khai;
- metric riêng trên CloudWatch, cơ chế thu thập log backend và năm cấu hình alarm; và
- Workshop song ngữ liên kết từng bước triển khai với ảnh chụp, kết quả kiểm thử và hướng dẫn xử lý sự cố.

Những điều chỉnh này giúp kiến trúc bám sát mã nguồn, phần cứng YOLO UNO và bài toán Smart Room của nhóm.

## Đóng góp cá nhân

| Thành viên | Phạm vi phụ trách và đóng góp cụ thể | Đường dẫn bằng chứng |
| :--- | :--- | :--- |
| **Phạm Lê Minh Khôi** | Kiến trúc AWS, ranh giới mạng/bảo mật và vận hành EC2/RDS/CloudWatch; phát triển firmware PlatformIO cho YOLO UNO; tích hợp cảm biến, LCD và thiết bị chấp hành; gửi telemetry, thăm dò lệnh, thực thi đầy đủ 8 lệnh và xử lý ACK | [Kiến trúc](../5.3-Architecture-and-Service-Design/), [thiết lập AWS](../5.4-AWS-Infrastructure-Setup/), [phần cứng](../5.6-Hardware-Integration/), [CloudWatch](../5.9-CloudWatch-Monitoring/) |
| **Ngô Minh Thuận** | Các route FastAPI, alias trong Pydantic, mô hình SQLAlchemy, lưu trữ bền vững bằng PostgreSQL, dịch vụ telemetry, vòng đời lệnh và xử lý ACK | [Thiết kế API/dữ liệu](../5.3-Architecture-and-Service-Design/), [backend/cơ sở dữ liệu](../5.5-Backend-and-Database/), [ma trận kiểm thử](../5.8-End-to-End-Testing/) |
| **Thượng Đình Hưng** | Dashboard React/Vite, trực quan hóa telemetry, yêu cầu điều khiển, giao diện chế độ/đề xuất, gỡ lỗi tích hợp và quay/dựng video minh họa | [tích hợp frontend](../5.7-Frontend-Integration/), [xác minh đầu cuối](../5.8-End-to-End-Testing/), [bàn giao](../5.12-Project-Handover/) |
| **Lê Bảo Khánh** | Nội dung đề xuất/báo cáo, blog/nhật ký/sự kiện, cấu trúc Workshop song ngữ, đối chiếu mã nguồn với tài liệu, điều hướng, kế hoạch ảnh và bảo đảm chất lượng | [tổng quan Workshop](../5.1-Workshop-overview/), [kế hoạch kiểm thử/bằng chứng](../5.8-End-to-End-Testing/), [kết quả](../5.11-Results-Challenges-Future/), [bàn giao](../5.12-Project-Handover/) |

Các mục Workshop được liên kết trong bảng là bằng chứng cho từng phạm vi phụ trách. Bảng này ghi lại vai trò và phần việc chính, không thay thế phần nhìn lại riêng của từng thành viên bên dưới.

## Nhìn lại theo từng thành viên (Individual Reflections)

### Phạm Lê Minh Khôi

| Nội dung nhìn lại | Trình bày |
| :--- | :--- |
| Challenge | Tích hợp backend phục vụ demo, PostgreSQL trong mạng riêng, hệ thống giám sát và thiết bị chấp hành vật lý mà vẫn phân biệt rõ thành công trên cloud với thành công của phần cứng |
| Root Cause | Luồng xử lý đi qua nhiều lớp: quy tắc VPC, IAM, dịch vụ Linux, cơ chế thăm dò HTTP, đấu nối phần cứng và trạng thái ACK bất đồng bộ |
| Solution | Dùng tham chiếu Security Group từ EC2 tới RDS, gắn EC2 IAM Role, kiểm tra `systemd` và CloudWatch, đối chiếu GPIO với mã nguồn, cấp nguồn an toàn, theo dõi ID lệnh và lưu trạng thái để gửi lại ACK |
| Lesson Learned | Cần xác minh từng ranh giới độc lập và theo dõi cùng một ID lệnh qua API, cơ sở dữ liệu, cổng nối tiếp, thao tác vật lý và hệ thống giám sát |
| Future Improvement | Bổ sung HTTPS và endpoint ổn định, chuẩn hóa cách định nghĩa hạ tầng, giới hạn quyền IAM chặt hơn, đồng thời cải thiện việc hiệu chuẩn cảm biến và lưu bằng chứng kiểm thử phần cứng |

### Ngô Minh Thuận

| Nội dung nhìn lại | Trình bày |
| :--- | :--- |
| Challenge | Lưu telemetry và giúp người vận hành theo dõi được toàn bộ quá trình hoàn tất lệnh qua cơ chế thăm dò và ACK |
| Root Cause | Các máy khách hoạt động bất đồng bộ nên trạng thái cơ sở dữ liệu có thể lệch; mã nguồn còn thiếu kiểm tra enum cho lệnh và chưa ràng buộc chặt ACK với thiết bị |
| Solution | Mô hình hóa thiết bị, telemetry và lệnh trong PostgreSQL; trả về ID cùng trạng thái lệnh; xử lý lệnh chờ theo FIFO và cập nhật trạng thái rõ ràng khi nhận ACK |
| Lesson Learned | Đặc tả OpenAPI và trạng thái được lưu giúp tăng khả năng truy vết, nhưng kiểm tra đầu vào, phân quyền, tính lũy đẳng và quy trình thay đổi lược đồ phải được thiết kế rõ |
| Future Improvement | Bổ sung kiểm tra lệnh được hỗ trợ, danh tính thiết bị đã xác thực, ACK gắn với thiết bị, quy tắc lũy đẳng, migration Alembic và kiểm thử API tự động |

### Thượng Đình Hưng

| Nội dung nhìn lại | Trình bày |
| :--- | :--- |
| Challenge | Hiển thị telemetry và điều khiển gần thời gian thực, đồng thời phân biệt rõ việc backend nhận yêu cầu, thiết bị thực thi lệnh và giao diện chuyển sang dữ liệu mô phỏng |
| Root Cause | Frontend thăm dò nhiều endpoint, chỉ lưu chế độ trên máy người dùng, chuyển sang dữ liệu mô phỏng khi lỗi và có thể báo thành công dù yêu cầu gửi lệnh thất bại |
| Solution | Kiểm tra thẻ Network của DevTools, dùng route API số nhiều với đường dẫn tương đối, hiển thị ID và trạng thái lệnh, gắn nhãn dữ liệu mô phỏng, đồng thời xác minh thao tác vật lý bằng ACK và bằng chứng |
| Lesson Learned | Giao diện phản hồi nhanh chưa đủ; trạng thái vận hành phải dựa trên backend và thiết bị, còn xử lý lỗi không được ám chỉ thành công khi chưa xác minh |
| Future Improvement | Loại bỏ cơ chế báo thành công giả, bổ sung chế độ/trạng thái lệnh từ API, tập trung cấu hình môi trường, sửa nhãn Lux và thêm kiểm thử component/tích hợp |

### Lê Bảo Khánh

| Nội dung nhìn lại | Trình bày |
| :--- | :--- |
| Challenge | Chuyển các ghi chú kỹ thuật và thay đổi trong quá trình triển khai thành một Workshop song ngữ mạch lạc |
| Root Cause | Backend, frontend, firmware, cấu hình AWS và bằng chứng được nhiều thành viên cập nhật vào những thời điểm khác nhau |
| Solution | Lấy mã nguồn đang hoạt động làm chuẩn kỹ thuật, đồng bộ cấu trúc Anh–Việt, đặt bằng chứng vào đúng bước và rà soát tên, đường dẫn, bảng cùng liên kết |
| Lesson Learned | Tài liệu kỹ thuật cần phân biệt rõ phần đã triển khai, hành vi mong đợi, kết quả quan sát, giới hạn và hướng cải tiến; đồng thời hai ngôn ngữ phải luôn được cập nhật cùng nhau |
| Future Improvement | Bổ sung kiểm tra tự động cho Hugo, liên kết, thông tin bí mật và tính tương ứng Anh–Việt; duy trì đặc tả API/GPIO có phiên bản; tổ chức vòng rà soát cuối với toàn bộ thành viên |

## Thách thức chính và bài học rút ra

| Vấn đề | Nguyên nhân gốc | Cách xử lý | Bài học |
| :--- | :--- | :--- | :--- |
| Kết nối EC2 với RDS mà vẫn giữ RDS trong mạng riêng | Backend cần truy cập PostgreSQL nhưng database không nên mở công khai | Cho phép cổng 5432 từ Security Group của EC2 thay vì dùng CIDR công khai | Nên cấp quyền theo đúng workload và cổng cần thiết |
| Đồng bộ trạng thái lệnh giữa API, cơ sở dữ liệu và thiết bị | Quá trình thăm dò và ACK diễn ra bất đồng bộ | Lưu ID cùng trạng thái lệnh, sau đó đối chiếu cùng ID từ lúc tạo đến khi ACK | API trả thành công chưa đủ để chứng minh thiết bị đã thực thi |
| Nguy cơ thực thi lệnh lặp khi thử lại | Thiết bị có thể nhận hoặc xác nhận cùng một lệnh nhiều lần | Theo dõi lệnh gần nhất và tách việc gửi lại ACK khỏi thao tác với thiết bị chấp hành | Cơ chế thử lại phải bảo đảm tính lũy đẳng |
| Route giữa frontend và backend không đồng nhất | Đường dẫn số ít/số nhiều và địa chỉ API thay đổi trong quá trình tích hợp | Dùng đặc tả OpenAPI đang triển khai và kiểm tra request trong DevTools Network | Cần duy trì một hợp đồng API có phiên bản |
| Giá trị ánh sáng chưa được hiệu chuẩn | Cảm biến trả về giá trị analog thô | Giữ giá trị gốc để truy vết và đưa việc hiệu chuẩn vào kế hoạch tiếp theo | Không gán đơn vị vật lý khi chưa có công thức quy đổi |
| Một số metric CloudWatch chưa có đủ datapoint | Quyền Agent, dimension, đường dẫn nguồn hoặc thời điểm thu thập có thể chưa khớp | Kiểm tra log Agent, dimension và đường dẫn nguồn thực tế | Phải đọc trạng thái alarm cùng với dữ liệu metric bên dưới |

## Giới hạn hiện tại

- Backend demo đang dùng HTTP qua cổng 8000 và địa chỉ công khai của EC2 có thể thay đổi sau khi dừng rồi khởi động lại instance.
- Các route API chưa áp dụng cơ chế xác thực mạnh cho máy khách và thiết bị.
- Backend cần kiểm tra chặt hơn danh sách lệnh được hỗ trợ và quan hệ giữa ACK với thiết bị.
- Frontend cần xử lý rõ ràng hơn khi chuyển sang dữ liệu mô phỏng hoặc khi request điều khiển thất bại.
- Giá trị ánh sáng hiện dựa trên dữ liệu analog chưa được hiệu chuẩn.
- Hai alarm CloudWatch ở trạng thái `Insufficient data`, đồng thời chưa có action thông báo tại thời điểm chụp.

## Cải tiến tương lai

- Sử dụng domain hoặc endpoint ổn định cho backend đã triển khai.
- Bổ sung reverse proxy phù hợp, HTTPS, xác thực và phân quyền chặt chẽ hơn.
- Lưu bí mật ứng dụng trong giải pháp quản lý bí mật phù hợp.
- Hỗ trợ nhiều thiết bị/phòng với quy tắc sở hữu và phân quyền.
- Bổ sung kiểm tra lệnh hợp lệ, ràng buộc ACK với thiết bị và quy tắc lũy đẳng.
- Áp dụng migration cơ sở dữ liệu có phiên bản và kiểm thử API tự động.
- Tập trung cấu hình môi trường của frontend và loại bỏ cơ chế báo thành công khi request thất bại.
- Chuẩn hóa cách định nghĩa hạ tầng, quy trình triển khai và phương án quay lui đã được kiểm thử.
- Bổ sung kênh thông báo phù hợp cho các alarm vận hành.
- Hiệu chuẩn cảm biến ánh sáng và công bố phương pháp cùng đơn vị quy đổi.

Mỗi hạng mục cải tiến cần có người phụ trách, kế hoạch triển khai và bằng chứng kiểm thử trước khi được mô tả là một phần của hệ thống hiện tại.

## Kết quả

Dự án đã hoàn thành phạm vi Smart Room đề ra: telemetry được thu thập và lưu trữ, dashboard hiển thị dữ liệu hiện tại cùng lịch sử, các lệnh điều khiển được thực thi trên ba thiết bị chấp hành, ACK cập nhật trạng thái lệnh và CloudWatch cung cấp bằng chứng vận hành. Cả T01–T12 đều được ghi nhận Đạt; các giới hạn nêu trên là cơ sở để xác định thứ tự ưu tiên cho giai đoạn cải tiến tiếp theo.

Tiếp theo: [chuẩn bị bàn giao dự án](../5.12-Project-Handover/).
