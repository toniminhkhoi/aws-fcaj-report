---
title: "Kiểm thử và xác minh đầu cuối"
date: "2026-07-28"
weight: 8
chapter: false
pre: " <b> 5.8. </b> "
---

## Bước 1 - Xác định chiến lược kiểm thử

Luồng lệnh được xác minh bằng cách đối chiếu bằng chứng từ nhiều lớp:

```text
Giao diện React + Vite
    -> Endpoint lệnh của FastAPI
    -> Bảng commands trong PostgreSQL
    -> YOLO UNO thăm dò lệnh
    -> Phản ứng của thiết bị chấp hành vật lý
    -> Yêu cầu ACK
    -> Trạng thái lệnh: Executed
```

Phòng mẫu được định danh bằng `device_id=room_01`. Khi có thể, cần ghi lại thời điểm kiểm thử và ID lệnh; đồng thời che thông tin xác thực, địa chỉ riêng và mã tài khoản trước khi công bố bằng chứng.

Không có ảnh chụp đơn lẻ nào chứng minh được toàn bộ luồng. Bằng chứng frontend/API xác nhận các yêu cầu từ trình duyệt, bằng chứng phần cứng xác nhận phản ứng vật lý, còn bằng chứng PostgreSQL xác nhận việc lưu lệnh và trạng thái của lệnh. Chỉ kết luận kiểm thử đầu cuối đạt khi đã đối chiếu các lớp bằng chứng này với nhau.

## Bước 2 - Xác minh các yêu cầu API từ frontend

1. Khởi động frontend React + Vite và mở trạm điều khiển.
2. Mở **Chrome DevTools > Network**, sau đó chọn bộ lọc **Fetch/XHR**.
3. Quan sát các yêu cầu `latest` và `history` được gửi định kỳ.
4. Thực hiện một thao tác điều khiển được hỗ trợ và xác nhận có yêu cầu `commands`.
5. Kiểm tra các yêu cầu hiển thị đều nhận phản hồi HTTP 200.

![Các yêu cầu API của frontend trong Chrome DevTools](/images/5-Workshop/5.8-validation/control-panel-api-request.png)

*Hình 15. Chrome DevTools xác nhận các request `latest`, `history` và `commands` từ frontend nhận phản hồi HTTP 200 từ FastAPI backend.*

Ảnh cho thấy telemetry trên dashboard cùng các yêu cầu XHR lặp lại đến `latest`, `history` và `commands`. Bằng chứng này xác nhận frontend giao tiếp thành công với backend trong quá trình thăm dò REST định kỳ; không dùng ảnh này để khẳng định một mức độ trễ cố định.

## Bước 3 - Kiểm thử điều khiển quạt

1. Chuyển thiết bị sang chế độ thủ công nếu hành vi firmware yêu cầu.
2. Gửi `FAN_ON` hoặc `FAN_OFF` từ dashboard.
3. Đối chiếu nút điều khiển đã chọn trên dashboard với phản ứng của quạt vật lý.
4. Chuyển lại chế độ tự động và xác nhận cơ chế điều khiển dựa trên luật tiếp tục hoạt động.
5. Kiểm tra lệnh liên quan đã được xác nhận bằng ACK.

![Đối chiếu điều khiển quạt trên dashboard với quạt vật lý](/images/5-Workshop/5.8-validation/dashboard-hardware-control-fan.png)

*Hình 16. Kiểm thử điều khiển quạt end-to-end: trạng thái trên dashboard được đối chiếu với phản ứng của quạt vật lý.*

Khung hình được dùng để đối chiếu trạng thái điều khiển trên dashboard với phản ứng quan sát được của quạt. Riêng ảnh này không chứng minh việc lưu lệnh trong cơ sở dữ liệu hoặc quá trình chuyển trạng thái sau ACK.

| ID | Phép kiểm thử | Kết quả mong đợi | Bằng chứng | Trạng thái |
| :--- | :--- | :--- | :--- | :---: |
| T01 | Gửi `FAN_ON` hoặc `FAN_OFF` | Quạt thay đổi trạng thái và lệnh được xác nhận | Hình 16, video demo và bản ghi lệnh | **Đạt** |
| T02 | Chuyển lại chế độ tự động | Thiết bị tiếp tục cơ chế điều khiển tự động dựa trên luật xác định | Trạng thái dashboard và video phần cứng | **Đạt** |

## Bước 4 - Kiểm thử điều khiển đèn

1. Gửi `LIGHT_ON` từ dashboard và quan sát đèn LED vật lý.
2. Gửi `LIGHT_OFF` và xác nhận đèn LED tắt.
3. Kiểm tra bản ghi lệnh tương ứng và xác nhận trạng thái cuối là `Executed`.

![Đối chiếu điều khiển đèn trên dashboard với đèn LED vật lý](/images/5-Workshop/5.8-validation/dashboard-hardware-control-led.png)

*Hình 17. Kiểm thử điều khiển đèn end-to-end: lệnh trên dashboard được xác nhận bằng trạng thái LED vật lý.*

| ID | Phép kiểm thử | Kết quả mong đợi | Bằng chứng | Trạng thái |
| :--- | :--- | :--- | :--- | :---: |
| T03 | Gửi `LIGHT_ON` | Đèn LED vật lý bật | Hình 17 và video demo | **Đạt** |
| T04 | Gửi `LIGHT_OFF` | Đèn LED vật lý tắt | Video demo phần cứng | **Đạt** |
| T05 | Kiểm tra trạng thái lệnh | Lệnh điều khiển đèn sau ACK có trạng thái `Executed` | Bản ghi lệnh trong PostgreSQL | **Đạt** |

## Bước 5 - Kiểm thử điều khiển rèm

1. Gửi `CURTAIN_OPEN` và quan sát chuyển động của servo.
2. Gửi `CURTAIN_CLOSE` và quan sát chuyển động theo chiều ngược lại.
3. Đối chiếu hai chuyển động với vị trí mở và đóng được cấu hình trong firmware.
4. Kiểm tra bản ghi lệnh tương ứng và xác nhận trạng thái cuối là `Executed`.

![Đối chiếu điều khiển rèm trên dashboard với servo vật lý](/images/5-Workshop/5.8-validation/dashboard-hardware-control-curtain.png)

*Hình 18. Kiểm thử điều khiển rèm end-to-end: lệnh MỞ/ĐÓNG trên dashboard được đối chiếu với chuyển động của servo vật lý.*

Tiêu chí nghiệm thu sử dụng các vị trí được cấu hình trong firmware; báo cáo không giả định một góc quay cụ thể.

| ID | Phép kiểm thử | Kết quả mong đợi | Bằng chứng | Trạng thái |
| :--- | :--- | :--- | :--- | :---: |
| T06 | Gửi `CURTAIN_OPEN` | Servo di chuyển đến vị trí mở đã cấu hình | Hình 18 và video demo | **Đạt** |
| T07 | Gửi `CURTAIN_CLOSE` | Servo di chuyển đến vị trí đóng đã cấu hình | Hình 18 và video demo | **Đạt** |
| T08 | Kiểm tra trạng thái lệnh | Lệnh điều khiển rèm sau ACK có trạng thái `Executed` | Bản ghi lệnh trong PostgreSQL | **Đạt** |

## Bước 6 - Xác minh trạng thái lệnh

Với mỗi thao tác được hỗ trợ, dashboard gửi lệnh đến FastAPI và backend lưu lệnh vào PostgreSQL. YOLO UNO thăm dò lệnh đang chờ, thực thi thao tác tương ứng trên thiết bị chấp hành rồi gọi endpoint ACK. Sau đó, backend chuyển trạng thái của lệnh từ `Pending` sang `Executed`.

Dùng các lệnh kiểm tra API và cơ sở dữ liệu dưới đây để đối chiếu cùng `device_id`, tên lệnh, ID và trạng thái:

```bash
curl -i http://127.0.0.1:8000/api/health
curl -s http://127.0.0.1:8000/api/devices/room_01/latest
curl -s http://127.0.0.1:8000/api/devices/room_01/history
```

```sql
SELECT id, device_id, command, state, timestamp
FROM commands
ORDER BY id DESC
LIMIT 6;
```

[Hình 9 trong mục 5.5](../5.5-Backend-and-Database/) là bằng chứng PostgreSQL cho lớp kiểm tra này. Ảnh hiển thị các lệnh gần nhất của `device_id=room_01`, gồm `CURTAIN_OPEN`, `CURTAIN_CLOSE`, `MODE_AUTO` và `LIGHT_OFF`, với trạng thái `Executed` sau khi được xác nhận.

Có thể xem thêm quá trình dashboard điều khiển phần cứng trong [video demo trên Google Drive](https://drive.google.com/file/d/1T97dUY58hbT2ppxvg7ESR12Jg9BA828W/view?usp=sharing). Các ảnh minh họa được cắt từ video nên có thể hơi mờ; video cung cấp đầy đủ hơn trình tự thao tác điều khiển.

## Bước 7 - Đối chiếu kết quả mong đợi

Phép kiểm thử được xem là đạt khi quan sát được đầy đủ các kết quả sau:

- Các yêu cầu `latest`, `history` và `commands` từ frontend nhận phản hồi HTTP 200.
- Dashboard hiển thị telemetry gắn với `device_id=room_01`.
- Quạt, đèn LED và servo rèm vật lý phản ứng với các lệnh được hỗ trợ.
- Thiết bị gửi ACK sau khi thực thi và trạng thái lệnh trong cơ sở dữ liệu chuyển thành `Executed`.
- Bằng chứng công bố không chứa thông tin xác thực, địa chỉ riêng hoặc mã tài khoản.
- Kết luận phân biệt rõ bằng chứng trình duyệt/API, bằng chứng phần cứng vật lý và bằng chứng trạng thái cơ sở dữ liệu.

## Xử lý sự cố

| Hiện tượng | Nội dung cần kiểm tra |
| :--- | :--- |
| `latest`, `history` hoặc `commands` không trả HTTP 200 | Kiểm tra Vite proxy/base URL, route FastAPI, dịch vụ backend và console của trình duyệt |
| Dashboard thay đổi nhưng thiết bị chấp hành không phản ứng | Kiểm tra chế độ thủ công/tự động, Wi-Fi của thiết bị, quá trình thăm dò lệnh, chính tả tên lệnh, dây nối và nguồn điện |
| Lệnh vẫn ở trạng thái `Pending` | Kiểm tra quá trình thăm dò của thiết bị, ID lệnh dùng cho endpoint ACK và log backend |
| Giao diện và PostgreSQL hiển thị khác trạng thái | Đối chiếu cùng một ID lệnh và tải lại bản ghi mới nhất sau ACK |
| Servo di chuyển không đúng | Kiểm tra vị trí mở/đóng được cấu hình và nguồn cấp cho servo |
| Bằng chứng chứa thông tin nhạy cảm | Che thông tin rồi chụp lại; thay mới bí mật đã lộ trước khi tiếp tục |

Tiếp theo: [cấu hình và xác minh CloudWatch](../5.9-CloudWatch-Monitoring/).