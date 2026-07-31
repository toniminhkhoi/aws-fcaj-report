---
title: "Kiểm thử và xác minh đầu cuối"
date: "2026-07-28"
weight: 8
chapter: false
pre: " <b> 5.8. </b> "
---

## Tổng quan và mục tiêu

Trước tiên, kiểm tra độc lập từng điểm kết nối, sau đó đối chiếu toàn bộ luồng telemetry và luồng lệnh. Trước khi kiểm thử, dùng FastAPI `/docs` hoặc `/openapi.json` để xác nhận schema của phiên bản đang được triển khai. Phòng mẫu được định danh bằng `device_id=room_01`.

Phần này kết hợp ma trận kiểm thử đầy đủ với bằng chứng từ trình duyệt, API, PostgreSQL và phần cứng vật lý. Mỗi bằng chứng đều được mô tả đúng phạm vi để tránh dùng một ảnh đơn lẻ cho những kết luận vượt quá nội dung thực sự hiển thị.

![Dashboard, bản ghi lệnh PostgreSQL và phần cứng vật lý](/images/5-Workshop/5.8-validation/end-to-end-system-overview.png)
*Hình 14a. Bằng chứng đầu cuối kết hợp dashboard, trạng thái lệnh trong cơ sở dữ liệu và mô hình YOLO UNO vật lý; từng lớp vẫn được kiểm tra riêng bên dưới.*

## Bước 1 - Thiết lập quy trình và chiến lược kiểm thử

1. Ghi ngày kiểm thử, người thực hiện, mã commit của ứng dụng, phiên bản firmware, AWS Region và `device_id`.
2. Che thông tin xác thực, địa chỉ riêng và mã tài khoản trong bằng chứng công bố.
3. Thu thập request/response, log, trạng thái SQL, đầu ra thiết bị, trạng thái dashboard và phản ứng vật lý có liên quan.
4. Ghi kết quả quan sát vào cột **Thực tế/bằng chứng**; chỉ đánh dấu **Đạt**, **Không đạt** hoặc **Chưa chạy** sau khi đã kiểm tra.
5. Đưa phần cứng và dịch vụ về trạng thái an toàn sau các phép thử lỗi.

Đối chiếu luồng lệnh qua các lớp sau:

```text
Giao diện React + Vite trên CloudFront
    -> CloudFront behavior /api/*
    -> Application Load Balancer
    -> Endpoint lệnh của FastAPI
    -> Bảng commands trong PostgreSQL
    -> YOLO UNO thăm dò lệnh
    -> Phản ứng của thiết bị chấp hành vật lý
    -> Yêu cầu ACK
    -> Trạng thái lệnh: Executed
```

Không có ảnh chụp đơn lẻ nào chứng minh được toàn bộ luồng. Bằng chứng frontend/API xác nhận request từ trình duyệt, bằng chứng phần cứng xác nhận phản ứng vật lý, còn bằng chứng PostgreSQL xác nhận việc lưu và trạng thái lệnh. Chỉ kết luận kiểm thử đầu cuối đạt khi đã đối chiếu các lớp bằng chứng liên quan với nhau.

## Bước 2 - Thực thi và ghi ma trận kiểm thử

| ID | Mục tiêu | Điều kiện trước | Các bước | Kết quả mong đợi | Thực tế/bằng chứng | Đạt/Không đạt |
| :--- | :--- | :--- | :--- | :--- | :--- | :---: |
| T01 | Kiểm tra backend | Dịch vụ backend đang hoạt động | Gửi `GET /api/health` | HTTP 200 và nội dung health check đúng định nghĩa | Hình 8 trong mục 5.5 và phản hồi health check | **Đạt** |
| T02 | Gửi telemetry | Đã biết schema OpenAPI và có thể kết nối RDS | POST một payload hợp lệ cho `device_id=room_01` | Phản hồi thành công và có bản ghi telemetry được lưu | Phản hồi telemetry cùng dữ liệu `latest`/`history` tương ứng | **Đạt** |
| T03 | Lấy telemetry mới nhất | Đã hoàn tất T02 | Gửi `GET /api/devices/room_01/latest` | Trả về bản ghi mới nhất của `room_01` | Hình 15 và các telemetry card trên dashboard | **Đạt** |
| T04 | Lấy lịch sử telemetry | Đã có nhiều bản ghi | Gửi `GET /api/devices/room_01/history` | Trả về lịch sử của `room_01` theo đúng thứ tự | Hình 14, Hình 15 và các biểu đồ lịch sử | **Đạt** |
| T05 | Tạo lệnh | Backend và cơ sở dữ liệu sẵn sàng | POST một lệnh được hỗ trợ | Tạo được lệnh có ID và trạng thái ban đầu `Pending` | Request `commands` trong Hình 15 và các bản ghi ở Hình 9 | **Đạt** |
| T06 | Thăm dò lệnh | YOLO UNO đang trực tuyến | Quan sát quá trình thăm dò sau T05 | Thiết bị nhận đúng lệnh và ID | Video demo phần cứng | **Đạt** |
| T07 | Điều khiển quạt | Quạt được đấu nối và cấp nguồn an toàn | Gửi `FAN_ON`, `FAN_OFF`, sau đó chuyển lại chế độ tự động | Quạt phản ứng, lệnh được ACK và cơ chế tự động dựa trên luật tiếp tục hoạt động | Hình 16 và video demo phần cứng | **Đạt** |
| T08 | Điều khiển đèn | Đèn LED/đèn được đấu nối và cấp nguồn an toàn | Gửi `LIGHT_ON`, sau đó `LIGHT_OFF` | Trạng thái vật lý của đèn khớp với cả hai lệnh | Hình 17 và video demo phần cứng | **Đạt** |
| T09 | Điều khiển rèm | Servo được đấu nối và cấp nguồn an toàn | Gửi `CURTAIN_OPEN`, sau đó `CURTAIN_CLOSE` | Servo di chuyển đến vị trí mở và đóng được cấu hình trong firmware | Hình 18 và video demo phần cứng | **Đạt** |
| T10 | Xác minh vòng đời ACK | Có lệnh từ T05–T09 | Quan sát request ACK và truy vấn cùng ID lệnh | Lệnh chuyển từ `Pending` sang `Executed` | Hình 9 và bản ghi lệnh sau ACK | **Đạt** |
| T11 | Xác minh dữ liệu trong PostgreSQL | Có phiên kết nối cơ sở dữ liệu | Truy vấn telemetry và lệnh sau khi tải lại API | Có thể truy vấn lại các bản ghi đã lưu | Hình 9 và kết quả truy vấn SQL lặp lại | **Đạt** |
| T12 | Xác minh log CloudWatch | CloudWatch Agent và quá trình thu thập log đã được cấu hình | Tạo một health request hoặc telemetry request | Sự kiện backend tương ứng xuất hiện trong log stream dự kiến | Bằng chứng log backend trong mục 5.9 | **Đạt** |
| T13 | Xác minh route production của trình duyệt | CloudFront distribution đã triển khai | Mở domain CloudFront và quan sát Fetch/XHR | Trang tải từ S3 và request `/api/*` trả HTTP 200 qua ALB origin | Bằng chứng CloudFront/API ở mục 5.4 và 5.7 | **Đạt** |
| T14 | Xác minh target backend | ALB và ASG đã triển khai | Kiểm tra target group và gọi `/api/health` qua ALB | Hai target ở hai Availability Zone đều Healthy và health call trả HTTP 200 | Hình 5c và Hình 8a | **Đạt** |

## Bước 3 - Xác minh các request API từ frontend

1. Khởi động frontend React + Vite và mở trạm điều khiển.
2. Mở **Chrome DevTools > Network**, sau đó chọn **Fetch/XHR**.
3. Quan sát các request `latest` và `history` được gửi định kỳ.
4. Thực hiện một thao tác điều khiển được hỗ trợ và xác nhận có request `commands`.
5. Kiểm tra các request hiển thị đều nhận phản hồi HTTP 200.

![Các request API của frontend trong Chrome DevTools](/images/5-Workshop/5.8-validation/control-panel-api-request.png)

*Hình 15. Chrome DevTools xác nhận các request `latest`, `history` và `commands` từ frontend nhận phản hồi HTTP 200 từ FastAPI backend.*

Ảnh cho thấy telemetry trên dashboard cùng các request XHR lặp lại đến `latest`, `history` và `commands`. Ở production, các request dùng domain CloudFront và behavior `/api/*` trước khi tới ALB. Bằng chứng này xác nhận frontend giao tiếp với backend trong quá trình thăm dò REST định kỳ; không dùng ảnh để khẳng định một mức độ trễ cố định.

## Bước 4 - Kiểm thử điều khiển quạt

1. Chuyển thiết bị sang chế độ thủ công nếu hành vi firmware yêu cầu.
2. Gửi `FAN_ON` và `FAN_OFF` từ dashboard.
3. Đối chiếu trạng thái dashboard với phản ứng của quạt vật lý.
4. Chuyển lại chế độ tự động và xác nhận cơ chế điều khiển xác định trước dựa trên luật tiếp tục hoạt động.
5. Kiểm tra lệnh liên quan đã được ACK.

![Đối chiếu điều khiển quạt trên dashboard với quạt vật lý](/images/5-Workshop/5.8-validation/dashboard-hardware-control-fan.png)

*Hình 16. Kiểm thử điều khiển quạt end-to-end: trạng thái trên dashboard được đối chiếu với phản ứng của quạt vật lý.*

Khung hình được dùng để đối chiếu thao tác trên dashboard với phản ứng vật lý quan sát được. Riêng ảnh này không chứng minh việc lưu lệnh trong cơ sở dữ liệu hoặc quá trình chuyển trạng thái sau ACK.

## Bước 5 - Kiểm thử điều khiển đèn

1. Gửi `LIGHT_ON` từ dashboard và quan sát đèn LED/đèn vật lý.
2. Gửi `LIGHT_OFF` và xác nhận đèn tắt.
3. Kiểm tra bản ghi lệnh tương ứng và xác nhận trạng thái cuối là `Executed`.

![Đối chiếu điều khiển đèn trên dashboard với đèn LED vật lý](/images/5-Workshop/5.8-validation/dashboard-hardware-control-led.png)

*Hình 17. Kiểm thử điều khiển đèn end-to-end: lệnh trên dashboard được xác nhận bằng trạng thái LED vật lý.*

## Bước 6 - Kiểm thử điều khiển rèm

1. Gửi `CURTAIN_OPEN` và quan sát chuyển động của servo.
2. Gửi `CURTAIN_CLOSE` và quan sát chuyển động theo chiều ngược lại.
3. Đối chiếu hai chuyển động với vị trí mở và đóng được cấu hình trong firmware.
4. Kiểm tra bản ghi lệnh tương ứng và xác nhận trạng thái cuối là `Executed`.

![Đối chiếu điều khiển rèm trên dashboard với servo vật lý](/images/5-Workshop/5.8-validation/dashboard-hardware-control-curtain.png)

*Hình 18. Kiểm thử điều khiển rèm end-to-end: lệnh MỞ/ĐÓNG trên dashboard được đối chiếu với chuyển động của servo vật lý.*

Tiêu chí nghiệm thu sử dụng các vị trí được cấu hình trong firmware; báo cáo không giả định một góc quay cụ thể.

## Bước 7 - Kiểm tra API, cơ sở dữ liệu và trạng thái lệnh

Từ EC2 Linux Bash, kiểm tra health của backend và việc truy xuất telemetry:

```bash
curl -i http://127.0.0.1:8000/api/health
curl -s http://127.0.0.1:8000/api/devices/room_01/latest
curl -s http://127.0.0.1:8000/api/devices/room_01/history
```

Tạo telemetry bằng các trường camelCase được mô tả trong mục 5.6. Tạo lệnh với `{ "command": "FAN_ON" }`. Trường Pydantic cũng có alias `Command`; do `populate_by_name=True`, tên viết thường `command` vẫn được chấp nhận. Bản ghi thiết bị thường được tạo sau telemetry request hợp lệ đầu tiên.

Trong PostgreSQL `psql`, kiểm tra các lệnh gần nhất:

```sql
SELECT
    id,
    device_id,
    command,
    state,
    timestamp
FROM commands
ORDER BY id DESC
LIMIT 6;
```

Thiết bị có thể thăm dò và ACK nhanh đến mức truy vấn sau đó không còn thấy `Pending`. Vì vậy, cần lưu phản hồi POST tạo lệnh khi còn trạng thái ban đầu, rồi truy vấn cùng ID sau ACK để xác nhận `Executed`.

![Cùng một lệnh chuyển từ Pending sang Executed](/images/5-Workshop/5.8-validation/command-pending-executed.png)
*Hình 18a. Bằng chứng API và PostgreSQL đối chiếu cùng một ID lệnh trước và sau ACK, cho thấy trạng thái chuyển từ `Pending` sang `Executed`.*

Dashboard gửi lệnh đến FastAPI và backend lưu lệnh vào PostgreSQL. YOLO UNO thăm dò lệnh đang chờ, thực thi thao tác tương ứng trên thiết bị chấp hành rồi gọi endpoint ACK. Sau đó, backend chuyển trạng thái lệnh từ `Pending` sang `Executed`.

[Hình 9 trong mục 5.5](../5.5-Backend-and-Database/) là bằng chứng PostgreSQL cho lớp kiểm tra này. Ảnh hiển thị các lệnh gần nhất của `device_id=room_01`, gồm `CURTAIN_OPEN`, `CURTAIN_CLOSE`, `MODE_AUTO` và `LIGHT_OFF`, ở trạng thái `Executed` sau khi được xác nhận.

Có thể xem thêm quá trình dashboard điều khiển phần cứng trong [video demo trên Google Drive](https://drive.google.com/file/d/1T97dUY58hbT2ppxvg7ESR12Jg9BA828W/view?usp=sharing). Các ảnh minh họa được cắt từ video nên có thể hơi mờ; video thể hiện đầy đủ hơn trình tự thao tác điều khiển.

## Kết quả mong đợi

Mỗi dòng T01–T14 có nội dung trong cột **Thực tế/bằng chứng** và trạng thái **Đạt**, **Không đạt** hoặc **Chưa chạy**. Một kết quả đầu cuối chỉ được xem là đạt khi đối chiếu được mã định danh thiết bị hoặc ID lệnh phù hợp qua CloudFront/ALB, API, PostgreSQL, firmware, dashboard, thiết bị chấp hành vật lý và log liên quan.

Kết quả nghiệm thu gồm phản hồi HTTP 200 cho `latest`, `history` và `commands`; telemetry của `device_id=room_01`; phản ứng vật lý của quạt, đèn LED/đèn và servo; ACK sau thực thi; cùng trạng thái lệnh cuối là `Executed`.

## Xử lý sự cố

Không dùng kế hoạch kiểm thử để tự tạo số liệu về độ trễ, thông lượng, tính sẵn sàng hoặc độ tin cậy. Với mỗi phép thử không đạt, cần ghi rõ lớp xảy ra lỗi, request hoặc log làm bằng chứng, người phụ trách, cách khắc phục và kết quả chạy lại.

| Hiện tượng | Nội dung cần kiểm tra |
| :--- | :--- |
| `latest`, `history` hoặc `commands` không trả HTTP 200 | Ở production, kiểm tra CloudFront `/api/*`, ALB target health, route FastAPI, backend log và browser console; ở local, kiểm tra Vite proxy |
| Không thấy `Pending` | Lưu phản hồi POST tạo lệnh, sau đó truy vấn cùng ID lệnh sau ACK |
| Dashboard thay đổi nhưng thiết bị chấp hành không phản ứng | Kiểm tra chế độ thủ công/tự động, Wi-Fi, quá trình thăm dò lệnh, chính tả tên lệnh, dây nối và nguồn điện |
| Giao diện và PostgreSQL hiển thị khác trạng thái | Đối chiếu cùng một ID lệnh và tải lại bản ghi mới nhất sau ACK |
| Lệnh bị lặp | So sánh ID lệnh và tách việc gửi lại ACK khỏi việc lặp lại thao tác trên thiết bị chấp hành |
| Servo di chuyển không đúng | Kiểm tra vị trí mở/đóng được cấu hình và nguồn cấp cho servo |
| Không tái tạo được phép kiểm thử | Ghi mã commit, Region, device ID, timestamp/múi giờ và điều kiện ban đầu chính xác |
| Bằng chứng chứa thông tin nhạy cảm | Che thông tin rồi chụp lại; thay mới bí mật đã lộ trước khi tiếp tục |

Tiếp theo: [cấu hình và xác minh CloudWatch](../5.9-CloudWatch-Monitoring/).
