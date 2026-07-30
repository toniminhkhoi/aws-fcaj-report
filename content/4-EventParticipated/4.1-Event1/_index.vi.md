---
title: "Sự kiện 1"
date: "2025-09-09"
weight: 1
chapter: false
pre: " <b> 4.1. </b> "
---

# Báo cáo về “First Cloud AI Journey Community Day”

### Mục đích của sự kiện

- Chia sẻ những hành trình nghề nghiệp thực tế, chuyển từ các vai trò CNTT cơ bản sang kỹ sư Cloud và DevOps.
- Khám phá việc triển khai các ứng dụng GraphRAG sử dụng Amazon Bedrock và Amazon Neptune.
- Hiểu các nguyên tắc vàng để làm việc nhóm hiệu quả và các công cụ cộng tác.
- Tìm hiểu cách xây dựng kiến trúc game nhiều người chơi trên đám mây kết nối các máy khách Godot với AWS WebSockets.
- Khám phá cách kết hợp AWS WAF với Học máy (Machine Learning) cho Hệ thống Phát hiện Xâm nhập Mạng (NIDS).
- Giới thiệu Docker như một công nghệ container hóa và so sánh nó với công nghệ ảo hóa truyền thống.

### Diễn giả

- **Trần Trung Vinh** – Quản trị viên Hệ thống (System Administrator) tại Central Retail Group
- **Việt Phát** – Sinh viên chuyên ngành AI tại Đại học Công nghệ Swinburne
- **Trương Huy Phước** – Diễn giả về Làm việc nhóm
- **Nguyễn Quốc Bảo** – Diễn giả về Multiplayer trên Đám mây
- **Lê Hoàng Gia Đại** – Sinh viên năm cuối Đại học HUTECH / AWS G3
- **Bảo Huỳnh** – Junior Cloud Native Developer tại Endava Vietnam / Nhà sáng lập ITea Lab

### Những Điểm Nổi Bật

## Nội dung chính

1. **Từ IT Helpdesk đến Senior Sysadmin & Cloud/DevOps**
   - Việc chuyển đổi sang Cloud đòi hỏi phải áp dụng một tư duy mới liên quan đến AWS, mở rộng quy mô linh hoạt (elastic scaling) và các dịch vụ được quản lý, thay thế cho các cấu hình thủ công tại chỗ (on-premise).
   - Cơ sở hạ tầng dưới dạng mã (IaC) sử dụng Terraform cho phép triển khai có thể lặp lại và kiểm soát phiên bản của cơ sở hạ tầng.
   - Văn hóa DevOps hiện đại bao gồm các pipeline CI/CD, Docker, tự động hóa và sự cộng tác chặt chẽ với các nhóm phát triển.

2. **Xây dựng Ứng dụng GraphRAG**
   - RAG truyền thống tăng cường cho các LLM với kiến thức bên ngoài, nhưng GraphRAG bổ sung thêm khả năng nhận thức mối quan hệ và suy luận đa bước (multi-hop reasoning) thông qua việc duyệt đồ thị (graph traversal).
   - **Hướng được quản lý hoàn toàn (Fully Managed Route):** Sử dụng Amazon Bedrock Knowledge Bases để phân chia (chunking), trích xuất thực thể và tạo embeddings, kết hợp với Amazon Neptune Analytics để lưu trữ đồ thị.
   - **Hướng tùy chỉnh (Custom Route):** Sử dụng LlamaIndex cho pipeline xử lý và Amazon Neptune để lưu trữ Knowledge Graph (Sơ đồ Tri thức) và thực thi các truy vấn Cypher.

3. **Nghệ thuật Làm việc nhóm hiệu quả**
   - Hiệu quả của làm việc nhóm được thúc đẩy bởi 4 Nguyên tắc vàng: Mục tiêu rõ ràng & chung, Đúng người đúng việc, Giao tiếp cởi mở & Lắng nghe tích cực, và Trách nhiệm cá nhân.
   - Các công cụ kỹ thuật số được đề xuất để quản lý nhóm và giao tiếp bao gồm Trello, ClickUp, Google Workspace, Slack và Discord.

4. **Multiplayer trên Đám mây (Godot + AWS)**
   - Các game theo lượt hoặc có phòng chờ (lobby) có thể được xây dựng hiệu quả bằng cách sử dụng WebSockets, cung cấp giao tiếp hai chiều (full-duplex) theo thời gian thực và đáng tin cậy.
   - Kiến trúc AWS tích hợp API Gateway WebSockets, AWS Lambda (Node.js) cho logic game và DynamoDB để lưu trữ trạng thái kết nối cùng các lựa chọn của người chơi.
   - Các thách thức bao gồm các kết nối cũ/bị đứt (GoneException) và chi phí quét DynamoDB; AWS GameLift được khuyến nghị cho việc đồng bộ hóa liên tục và quản lý trạng thái game có thẩm quyền trong bộ nhớ.

5. **NIDS dựa trên Học máy (Machine Learning) trên AWS**
   - AWS WAF truyền thống dựa trên các quy tắc được định nghĩa trước, vốn gặp khó khăn trước các kỹ thuật tấn công mới, zero-day hoặc kết hợp (hybrid).
   - Một mô hình Học máy được đào tạo trên tập dữ liệu CSE-CIC-IDS2018 có thể chủ động phát hiện các hành vi bất thường chưa từng có.
   - Kiến trúc hệ thống tích hợp AWS WAF, Amazon EC2, Application Load Balancer và các mô hình Học máy như LightGBM và Random Forest để đạt độ chính xác cao.

6. **Docker và Công nghệ Container hóa**
   - Công nghệ container hóa đóng gói một ứng dụng với tất cả các thành phần phụ thuộc của nó để nó chạy nhất quán trên bất kỳ môi trường nào (vật lý, ảo hóa hoặc đám mây).
   - Không giống như Máy ảo (Virtual Machines) vốn cồng kềnh, yêu cầu hệ điều hành riêng và mất nhiều phút để khởi động, các container Docker chia sẻ hệ điều hành máy chủ, khởi động tính bằng mili-giây và sử dụng ít tài nguyên hơn.
   - Docker sử dụng các lớp (layers) từ Dockerfile; các lớp không thay đổi được sử dụng lại từ bộ nhớ đệm (cache) trong quá trình build để tăng tốc quá trình này.

### Những Bài học Quan trọng

- **Tư duy Thiết kế & Vận hành**
  - Không bao giờ thử nghiệm trên môi trường production—hãy bảo vệ tính khả dụng, niềm tin và thời gian của nhóm bạn.
  - Tự động hóa các tác vụ lặp đi lặp lại để tiết kiệm thời gian, đồng thời lập tài liệu rõ ràng cho các cấu hình và runbook (sổ tay vận hành).
  - Một portfolio thực tế và kinh nghiệm thực hành có giá trị hơn nhiều so với việc chỉ sở hữu các chứng chỉ.

- **Kiến trúc Kỹ thuật**
  - Khi xây dựng các game multiplayer không máy chủ (serverless), hãy lưu ý rằng AWS Lambda là phi trạng thái (stateless), nghĩa là trạng thái game phải được lưu và truy xuất từ DynamoDB trong mỗi yêu cầu.
  - Việc chỉ sử dụng bảo vệ dựa trên chữ ký (signature-based) là không đủ đối với bảo mật hiện đại; NIDS dựa trên ML bổ sung hiệu quả cho các tường lửa truyền thống như AWS WAF.
  - Việc tận dụng công nghệ container (Docker) là lý tưởng cho các pipeline CI/CD, kiến trúc microservices và hiện đại hóa các ứng dụng cũ.

- **Chiến lược Dữ liệu & AI**
  - Trong Học máy, chất lượng dữ liệu là rất quan trọng; việc xử lý sự mất cân bằng dữ liệu là cần thiết để cải thiện khả năng phát hiện các lớp tấn công thiểu số.
  - Các phương pháp tiếp cận GraphRAG vượt qua những hạn chế của LLM tiêu chuẩn bằng cách cung cấp khả năng lưu trữ mối quan hệ rõ ràng thông qua các nút (nodes) và cạnh (edges).

### Ứng dụng vào Công việc

- **Trong cơ sở hạ tầng & vận hành**:
  - Áp dụng Cơ sở hạ tầng dưới dạng mã (Terraform) để định nghĩa cơ sở hạ tầng ảo và cơ sở dữ liệu thay vì thực hiện các cấu hình thủ công.
  - Triển khai giám sát hệ thống trước khi sự cố xảy ra và duy trì tư duy vận hành tập trung vào việc phòng ngừa.

- **Trong phát triển phần mềm & AI**:
  - Đóng gói các ứng dụng bằng Docker để loại bỏ các vấn đề tương thích kiểu "nó chạy ngon trên máy tôi".
  - Thử nghiệm với Amazon Bedrock và Neptune để trích xuất các thực thể và xây dựng Sơ đồ Tri thức (Knowledge Graphs) cho các phản hồi LLM chính xác hơn.

- **Trong làm việc nhóm**:
  - Áp dụng nguyên tắc "Đúng người, đúng việc" và tận dụng các công cụ như ClickUp hoặc Slack để tối ưu hóa việc theo dõi dự án và giao tiếp.

#### Hình ảnh Sự kiện

<img src="/images/4-EventParticipated/image_1.jpg" alt="Event 1" width="600"/>