---
title: "Tích hợp phần cứng"
date: "2026-07-28"
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

## Tổng quan và mục tiêu

YOLO UNO là thiết bị chính của Workshop. Thiết bị đọc nhiệt độ, độ ẩm từ DHT20 và giá trị ánh sáng analog thô; điều khiển quạt, đèn/relay và servo rèm; gửi telemetry; định kỳ kiểm tra lệnh đang chờ; thực thi mỗi lệnh một lần rồi gửi ACK.

## Giới thiệu về YOLO UNO

YOLO UNO là bo mạch phát triển có kiểu dáng và cách bố trí chân tương thích với Arduino Uno truyền thống, nhưng sử dụng vi điều khiển **ESP32-S3** của Espressif. Bo mạch có **16 MB Flash** và tối đa **8 MB PSRAM**, phù hợp với các ứng dụng nhúng, IoT và xử lý dữ liệu ngay trên thiết bị.

Ngoài kích thước và cách bố trí chân tương thích với Arduino Uno, YOLO UNO còn tích hợp **12 cổng Grove**. Nhờ đó, người dùng có thể kết nối nhanh cảm biến và thiết bị chấp hành mà không cần breadboard hoặc quá nhiều dây jumper.

Trong dự án **AWS IoT Monitoring and Control Dashboard**, YOLO UNO thực hiện các nhiệm vụ:

- đọc nhiệt độ và độ ẩm từ DHT20;
- đọc giá trị ADC thô từ cảm biến ánh sáng analog;
- điều khiển quạt hai chân;
- điều khiển đèn LED hoặc relay;
- điều khiển servo đóng/mở rèm;
- hiển thị trạng thái trên LCD 1602 I2C;
- kết nối Wi-Fi và gửi telemetry đến FastAPI backend; và
- thăm dò lệnh từ backend, thực thi lệnh rồi gửi ACK.

Phần lớn cảm biến và thiết bị chấp hành kết nối qua các cổng Grove ở giữa bo mạch. Riêng servo rèm dùng cổng ba chân **GVS D11**, tương ứng với **GPIO38** trong firmware.

Mô hình dưới đây cho thấy YOLO UNO cùng màn hình, cảm biến, quạt và servo điều khiển rèm được sử dụng trong dự án.

![Mô hình phần cứng YOLO UNO với cảm biến và actuator](/images/5-Workshop/5.6-hardware/yolo-uno-hardware-setup.png)
*Hình 10. Mô hình phần cứng thực tế gồm YOLO UNO, màn hình LCD, cảm biến nhiệt độ và độ ẩm, cảm biến ánh sáng, quạt và servo điều khiển rèm.*

> **Ghi chú:** Hình ảnh được trích từ video nên một số chi tiết có thể hơi mờ. Bạn có thể xem [video demo đầy đủ trên Google Drive](https://drive.google.com/file/d/1T97dUY58hbT2ppxvg7ESR12Jg9BA828W/view?usp=sharing) để quan sát rõ hơn.

## Bước 1 - Nối phần cứng theo mã nguồn

Firmware hiện dùng trong `hardware/src/main.cpp` định nghĩa sơ đồ chân dưới đây. Khi có khác biệt, các giá trị trong file này được ưu tiên hơn những mô tả cũ ở nơi khác trong kho mã nguồn.

Ảnh pinout YOLO UNO chưa được cung cấp. Pinout không thay thế sơ đồ đấu dây hoàn chỉnh; hãy đối chiếu bảng đã xác minh dưới đây với các định nghĩa trong firmware.

<!-- TODO: Add yolo-uno-pinout-gpio-mapping.png -->

### Ánh xạ cổng phần cứng

| Thiết bị | Cổng vật lý trên YOLO UNO | Chân trong firmware |
| :--- | :--- | :--- |
| Cảm biến ánh sáng analog | Grove `A1-A0` | `A0 / GPIO1` |
| Quạt hai chân điều khiển | Grove `D8-D7` | `D8 / GPIO17`, `D7 / GPIO10` |
| Đèn LED hoặc relay | Grove `D4-D3` | `D3 / GPIO6` |
| Servo rèm | Cổng GVS `D11` | `D11 / GPIO38` |
| DHT20 | Grove `I2C1` | `SDA / GPIO11`, `SCL / GPIO12` |
| LCD 1602 I2C | Grove `I2C2` | `SDA / GPIO11`, `SCL / GPIO12` |

DHT20 được cắm vào Grove `I2C1`, còn LCD 1602 dùng Grove `I2C2`. Hai cổng dùng chung bus I2C với SDA/GPIO11 và SCL/GPIO12. DHT20 dùng địa chỉ `0x38`; firmware tự dò LCD ở `0x21`, `0x27` hoặc `0x3F`. Nếu LCD cần 5 V, phải dùng bộ chuyển mức logic hai chiều và không nối pull-up 5 V trực tiếp vào GPIO của ESP32-S3. Hai module chỉ dùng chung bus, không nối trực tiếp với nhau bằng dây ngoài.

Firmware không sử dụng PIR, cảm biến siêu âm, buzzer hoặc MQTT. Hãy dùng nguồn phù hợp cho thiết bị chấp hành, nối chung mass và không cấp dòng trực tiếp từ GPIO cho quạt hoặc servo.

## Bước 2 - Chuẩn bị PlatformIO

Mở dự án phần cứng trong VS Code và xác nhận:

- có file JSON định nghĩa bo mạch YOLO UNO / ESP32-S3 và được tham chiếu đúng;
- `platformio.ini` chọn đúng môi trường và thư viện;
- baud Serial Monitor là `115200`;
- môi trường là `yolo_uno` trên ESP32-S3;
- các thư viện phụ thuộc ArduinoJson, ESP32Servo, DHT20 và LiquidCrystal_I2C được tải thành công;
- `include/secrets.example.h` được commit; và
- `include/secrets.h` chỉ tồn tại cục bộ và đã được loại khỏi Git.

Dùng cấu trúc file bí mật sau:

```cpp
#pragma once
constexpr char WIFI_SSID[] = "<YOUR_WIFI_SSID>";
constexpr char WIFI_PASSWORD[] = "<YOUR_WIFI_PASSWORD>";
constexpr char API_BASE_URL[] = "http://<ALB_DNS_NAME>";
constexpr char DEVICE_ID[] = "room_01";
```

Không công khai mật khẩu Wi-Fi thật. Route thiết bị đã kiểm chứng là HTTP trực tiếp tới DNS của ALB; không đưa IP của EC2 hoặc domain CloudFront của frontend vào firmware, trừ khi có một route thiết bị khác đã được kiểm thử riêng.

## Bước 3 - Xác minh cảm biến và thiết bị chấp hành tại chỗ

1. Khởi tạo I2C và xác nhận DHT20 phản hồi.
2. Đọc nhiệt độ, độ ẩm; loại bỏ giá trị không hợp lệ hoặc NaN.
3. Đọc và ghi nhận **giá trị ánh sáng analog thô** cho đến khi mã nguồn có phép quy đổi đã hiệu chuẩn.
4. Khởi tạo đầu ra quạt và đèn ở trạng thái an toàn.
5. Gắn servo và kiểm tra vị trí đóng ở 0°, mở ở 90°.
6. Xác nhận LCD được tìm thấy ở một trong ba địa chỉ hỗ trợ.
7. Xác nhận lỗi cảm biến không làm bo mạch khởi động lại liên tục hoặc chặn vòng lặp xử lý lệnh.

Dùng mạch điều khiển và diode bảo vệ ngược cho tải cảm ứng khi cần. Không cấp dòng cho servo hoặc quạt trực tiếp từ GPIO.

## Bước 4 - Gửi telemetry

Firmware tạo JSON với đúng các tên trường camelCase mà backend chấp nhận:

```json
{
  "deviceId": "room_01",
  "temperature": 25.0,
  "humidity": 60.0,
  "lightIntensity": 512,
  "fan": false,
  "light": false,
  "curtain": false
}
```

`lightIntensity` là giá trị analog thô. Mã nguồn gửi telemetry mỗi 5000 ms, thăm dò lệnh mỗi 2000 ms, cập nhật LCD mỗi 2000 ms, thử kết nối lại Wi-Fi mỗi 10000 ms với thời gian chờ kết nối 20000 ms và thời gian chờ HTTP 7000 ms.

```text
YOLO UNO đọc cảm biến → tạo JSON → POST /api/telemetry → kiểm tra trạng thái HTTP → chờ đến chu kỳ tiếp theo
```

ALB forward các request từ thiết bị tới một FastAPI target Healthy trên cổng 8000. Thiết bị không chọn và không phụ thuộc vào một instance cụ thể trong ASG.

Khi HTTP trả về trạng thái không thành công, hãy ghi lại phản hồi và thử lại sau một khoảng trễ có giới hạn. Việc gửi dữ liệu qua mạng không được chặn vô thời hạn logic an toàn của thiết bị chấp hành.

## Bước 5 - Thăm dò, thực thi và xác nhận lệnh

Thăm dò lệnh:

```text
GET /api/devices/room_01/commands/latest
```

Firmware chấp nhận tám lệnh: `MODE_AUTO`, `MODE_MANUAL`, `FAN_ON`, `FAN_OFF`, `LIGHT_ON`, `LIGHT_OFF`, `CURTAIN_OPEN`, `CURTAIN_CLOSE`. Lệnh điều khiển trực tiếp sẽ chuyển firmware sang chế độ thủ công. Trong chế độ tự động, mã nguồn bật quạt khi nhiệt độ ≥30°C, bật đèn khi giá trị analog <350 và mở rèm khi giá trị analog <700.

```cpp
if (pending && commandId != lastExecutedCommandId) {
  const bool applied = applySupportedCommand(command);
  if (applied) {
    lastExecutedCommandId = commandId;
    sendAck(commandId);
  }
}
```

ACK:

```text
POST /api/devices/room_01/commands/{command_id}/ack
```

Nếu gửi ACK thất bại sau khi thiết bị chấp hành đã hoạt động, chỉ thử gửi lại ACK, không thực thi lại thao tác. Mã nguồn lưu `autoMode`, `lastAck`, `pendingAck` trong ESP32 Preferences để có thể gửi lại ACK và tránh chạy lại cùng một lệnh sau khi bo mạch khởi động lại. Firmware từ chối lệnh không được hỗ trợ và không gửi ACK.

## Bước 6 - Biên dịch, nạp firmware và theo dõi

Trong PlatformIO terminal:

```powershell
pio run -e yolo_uno
```

Việc build firmware không yêu cầu bo mạch phải kết nối qua USB. Khi build thành công, PlatformIO tạo `firmware.bin` và kết thúc bằng:

```text
SUCCESS
```

Ảnh dưới đây cho thấy firmware đã được biên dịch thành công.

![PlatformIO build firmware YOLO UNO thành công](/images/5-Workshop/5.6-hardware/platformio-firmware-build.png)
*Hình 12. Firmware YOLO UNO được biên dịch thành công bằng PlatformIO trong environment `yolo_uno`, tạo file `firmware.bin` với kết quả `SUCCESS`.*

Chỉ có thể nạp firmware và mở Serial Monitor khi YOLO UNO đã kết nối với máy tính:

```powershell
pio run --target upload
pio device monitor --baud 115200
```

Khi bo mạch đã kết nối và firmware đang chạy, Serial Monitor dự kiến hiển thị chuỗi thông báo sau:

```text
[wifi] connected
[telemetry] HTTP success for room_01
[command] pending command received: <COMMAND_ID> <SUPPORTED_COMMAND>
[actuator] command applied once
[ack] command acknowledged
```

## Kết quả mong đợi

PlatformIO biên dịch thành công môi trường `yolo_uno` và tạo `firmware.bin`. Sau khi kết nối bo mạch và hoàn tất các bước tích hợp còn lại, YOLO UNO đọc DHT20 cùng cảm biến ánh sáng analog, cập nhật LCD1602, gửi telemetry đúng lược đồ, thực thi mỗi lệnh được hỗ trợ một lần và dùng ACK để chuyển lệnh tương ứng trên backend từ `Pending` sang `Executed`. Ảnh Serial Monitor không được để lộ mật khẩu Wi-Fi hoặc endpoint công khai.

## Xử lý sự cố

| Hiện tượng | Nội dung cần kiểm tra |
| :--- | :--- |
| Không tìm thấy DHT20 | Chân SDA/SCL, địa chỉ, nguồn và quá trình khởi tạo I2C |
| Giá trị ánh sáng kẹt min/max | Khả năng ADC của pin, dải điện áp, dây |
| Bo mạch khởi động lại khi bật thiết bị chấp hành | Nguồn ngoài, dòng tiêu thụ, diode bảo vệ và nối chung mass |
| Wi-Fi lặp lại quá trình kết nối | SSID/mật khẩu, cường độ tín hiệu, khoảng trễ chặn và thời gian chờ giữa các lần thử |
| HTTP hết thời gian chờ | DNS ALB, Wi-Fi/DNS, listener HTTP:80, target health, backend SG và địa chỉ bind của Uvicorn |
| Lệnh bị lặp | So sánh ID lệnh và tách việc thực thi thiết bị khỏi việc gửi lại ACK |
| Lệnh giữ trạng thái `Pending` | URL/ID/nội dung ACK, phản hồi HTTP và log backend |
| Lệnh không được hỗ trợ | Ghi log và từ chối; không ACK thành `Executed` |

Tiếp theo: [kết nối React dashboard](../5.7-Frontend-Integration/).
