---
title: "Tích hợp frontend"
date: "2026-07-28"
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

## Tổng quan và mục tiêu

Chạy React + Vite frontend trên máy cục bộ, kết nối với FastAPI trên EC2 qua HTTP REST, đồng thời hiển thị telemetry được làm mới định kỳ, bảng điều khiển, đề xuất dựa trên luật và lịch sử của phòng mẫu có `device_id = room_01`.

## Bước 1 - Cấu hình React frontend

Dự án sử dụng React, Vite, TypeScript, Tailwind CSS, Axios và Recharts. Từ Windows PowerShell:

```powershell
git clone <REPOSITORY_URL>
Set-Location .\aws-iot-dashboard\frontend
npm install
npm run dev
```

Sử dụng phiên bản Node theo `package.json` và giữ nguyên lockfile của repository. Frontend chạy cục bộ ngoài AWS.

## Bước 2 - Kết nối frontend với FastAPI backend

Dùng đường dẫn tương đối `/api` với Vite development proxy:

```ts
// vite.config.ts
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";

export default defineConfig({
  plugins: [react()],
  server: {
    proxy: {
      "/api": {
        target: "http://<EC2_PUBLIC_IP>:8000",
        changeOrigin: true,
      },
    },
  },
});
```

Khởi động lại Vite sau khi đổi cấu hình. Nếu dự án dùng `VITE_API_BASE_URL`, lưu biến trong `.env.local` đã được loại khỏi Git thay vì lặp URL EC2 trong nhiều component.

Các request chính gồm:

```text
GET  /api/health
GET  /api/devices/room_01/latest
GET  /api/devices/room_01/history
POST /api/devices/room_01/commands
```

Badge **LIVE AWS** phải dựa trên phản hồi backend/API thành công, không chỉ dựa vào việc trang React đã tải.

## Bước 3 - Hiển thị telemetry gần thời gian thực

Hiển thị ba card nhiệt độ, độ ẩm và cường độ ánh sáng. Frontend làm mới dữ liệu định kỳ bằng REST polling, vì vậy dashboard được mô tả là **gần thời gian thực**, không phải hệ thống thời gian thực có độ trễ cố định. Khi phù hợp, giao diện cần có trạng thái đang tải, lỗi có thể thử lại và thời điểm cập nhật gần nhất.

## Bước 4 - Hiển thị bảng điều khiển từ xa

Bảng điều khiển hỗ trợ:

- `FAN_ON` / `FAN_OFF`;
- `LIGHT_ON` / `LIGHT_OFF`;
- `CURTAIN_OPEN` / `CURTAIN_CLOSE`; và
- chuyển giữa `MODE_MANUAL` và `MODE_AUTO`.

Vô hiệu hóa nút đang gửi request, tránh tạo command trùng đang chờ và phân biệt việc backend nhận command với việc phần cứng thực thi. ID/trạng thái do backend trả về được dùng để theo dõi thay vì chỉ dựa vào trạng thái cục bộ của UI.

<p align="center">
  <img src="/images/5-Workshop/5.7-frontend/dashboard-overview-control-panel.png"
       alt="React Vite IoT dashboard hiển thị telemetry và bảng điều khiển thiết bị"
       width="100%" />
</p>

*Hình 13. React + Vite dashboard hiển thị telemetry gần thời gian thực và bảng điều khiển quạt, đèn và rèm cho phòng mẫu có `device_id = room_01`.*

Hình 13 cho thấy giao diện React + Vite chạy cục bộ, nhãn stack EC2 FastAPI/RDS PostgreSQL/React Vite, ba telemetry card có badge **LIVE AWS**, cùng bảng điều khiển quạt, đèn, rèm và chế độ. UI có thể hiển thị Manual Override hoặc Auto tùy trạng thái hiện tại.

## Bước 5 - Hiển thị phân tích dựa trên luật và lịch sử

### Phân tích và đề xuất dựa trên luật

Panel đề xuất đánh giá các điều kiện cố định dựa trên nhiệt độ, độ ẩm, ánh sáng và thời gian. Ví dụ trong giao diện gồm đề xuất tắt quạt ngoài giờ làm việc, thông báo độ ẩm trong khoảng phù hợp và đề xuất điều chỉnh rèm khi ánh sáng cao. Đây là phân tích xác định trước bằng luật/ngưỡng, không phải machine learning, predictive analytics hoặc mô hình đã được huấn luyện.

Các biểu đồ lịch sử hiển thị nhiệt độ, độ ẩm và ánh sáng được truy xuất từ Amazon RDS qua endpoint:

```text
GET /api/devices/room_01/history
```

<p align="center">
  <img src="/images/5-Workshop/5.7-frontend/dashboard-analysis-history.png"
       alt="Đề xuất dựa trên luật và biểu đồ lịch sử telemetry"
       width="100%" />
</p>

*Hình 14. Panel phân tích theo luật và biểu đồ lịch sử nhiệt độ, độ ẩm và ánh sáng được truy xuất từ Amazon RDS.*

## Bước 6 - Kết quả mong đợi

- React + Vite frontend tải thành công trên máy cục bộ.
- Badge **LIVE AWS** phản ánh một request backend/API thành công.
- Telemetry của `device_id = room_01` xuất hiện trên ba card.
- Các nút quạt, đèn, rèm và chế độ được hiển thị.
- Biểu đồ lịch sử nhận bản ghi từ history endpoint.
- Nội dung đề xuất được mô tả là dựa trên luật/ngưỡng, không phải machine learning.
- Telemetry được mô tả là gần thời gian thực qua REST polling, không kèm tuyên bố độ trễ thiếu bằng chứng.

## Xử lý sự cố

| Hiện tượng | Nội dung cần kiểm tra |
| :--- | :--- |
| Vite proxy trả 404 | Kiểm tra proxy key, target, route API số nhiều và khởi động lại Vite |
| Trình duyệt báo CORS | Xác nhận request đi qua proxy hoặc rà chính sách CORS của backend |
| Telemetry card trống | Kiểm tra `/api/health`, cấu trúc latest response, `device_id` và cách xử lý loading/error |
| Biểu đồ lịch sử trống | Kiểm tra history response, timestamp và cách xử lý mảng rỗng |
| Trạng thái luôn online | Liên kết badge với health/API response thực, không dựa vào thời điểm component được gắn |
| Command bị lặp | Vô hiệu hóa nút đang gửi và kiểm tra command cùng loại còn `Pending` hay không |
| UI báo thành công quá sớm | Tách trạng thái backend nhận request khỏi ACK/`Executed` và phản ứng vật lý |

Tiếp theo: [chạy kiểm thử end-to-end](../5.8-End-to-End-Testing/).
