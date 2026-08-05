---
title: "4. Cấu hình ALB & API Gateway"
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

# Cấu hình API Gateway, VPC Link V2 & Internal ALB

Hiện tại, máy chủ EC2 backend của bạn đã bị ẩn hoàn toàn khỏi mạng internet. Chúng ta sẽ tiến hành xây dựng một cây cầu bảo mật (VPC Link) cho phép API Gateway công cộng giao tiếp được với backend ẩn bên trong VPC thông qua một Bộ cân bằng tải nội bộ (Internal Load Balancer).

---

### Bước 1: Tạo Security Group cho ALB

1. Trong giao diện EC2 Dashboard, vào mục **Security Groups** và bấm **Create security group**.
2. **Name:** `pedix-alb-sg`.
3. **Description:** Security group for Internal ALB.
4. **VPC:** Chọn Default VPC.
5. **Inbound Rules:** Thêm 1 rule: Type `HTTP` (Cổng 80), Source `Anywhere-IPv4` (`0.0.0.0/0`).
6. Bấm **Create security group**.

> [!CAUTION]
> **Security Group Chaining:** Bây giờ bạn bắt buộc phải quay lại `pedix-ec2-sg` (của máy chủ EC2), chỉnh sửa Inbound Rules ở Cổng 8000: đổi Source (Nguồn) thành `pedix-alb-sg`. Thao tác này đảm bảo EC2 của bạn *chỉ* nhận dữ liệu đi ra từ Load Balancer!

---

### Bước 2: Tạo Target Group

1. Cũng ở giao diện EC2, cuộn thanh menu trái xuống mục **Target Groups** và bấm **Create target group**.
2. **Target type:** Chọn **Instances**.
3. **Target group name:** `pedix-backend-tg`.
4. **Protocol:** `HTTP`, **Port:** `8000`.
5. **VPC:** Chọn Default VPC.
6. **Health checks:** Đổi Health check path thành `/api/health`.
7. Bấm **Next**.
8. Tích chọn máy chủ `Pedix-Backend-Server` của bạn, bấm nút **Include as pending below**, rồi bấm **Create target group**.

---

### Bước 3: Tạo Internal Application Load Balancer

1. Trong menu EC2, bấm **Load Balancers** > **Create load balancer**.
2. Bấm **Create** ở mục **Application Load Balancer**.
3. **Load balancer name:** `pedix-internal-alb`.
4. **Scheme:** Chọn **Internal** (Cực kỳ quan trọng! Không được chọn Internet-facing).
5. **Network mapping:** Tích chọn tất cả các vùng (Availability Zones như `1a`, `1b`, `1c`).
6. **Security groups:** Chọn `pedix-alb-sg`.
7. **Listeners and routing:** Protocol `HTTP`, Port `80`, Forward to `pedix-backend-tg`.
8. Bấm **Create load balancer**.

---

### Bước 4: Tạo VPC Link V2 (Kết nối API Gateway vào VPC)

1. Tìm dịch vụ **API Gateway** trên thanh tìm kiếm AWS.
2. Ở menu trái, chọn **VPC links** > **Create**.
3. **VPC link version:** Chọn **VPC link for REST APIs**.
4. **Name:** `pedix-vpclink`.
5. **Target Load Balancer:** Chọn `pedix-internal-alb` vừa tạo.
6. Bấm **Create**. Sẽ mất khoảng 2-3 phút để trạng thái chuyển sang màu xanh *Available*.

---

### Bước 5: Cấu hình API Gateway & Định tuyến SSE Streaming

1. Trong console của API Gateway, bấm **APIs** > **Create API**.
2. Tìm mục **REST API** (cái *không* có chữ Private) và bấm **Build**.
3. **API name:** `Pedix-API`. Endpoint Type: `Regional`. Bấm **Create API**.

#### Tạo đường dẫn gom tất cả (Catch-All `/{proxy+}`)
1. Bấm **Create resource**. Bật tính năng **Proxy resource**, để nguyên Resource path là `/{proxy+}`. Bấm **Create resource**.
2. Bấm vào phương thức `ANY` vừa sinh ra bên dưới `/{proxy+}`.
3. **Integration type:** Chọn **VPC Link**.
4. Bật **Use proxy integration**.
5. **Method:** `ANY`.
6. **VPC Link:** Chọn `pedix-vpclink`.
7. **Endpoint URL:** `http://internal-pedix-internal-alb-[id].ap-southeast-1.elb.amazonaws.com/api/{proxy}` *(Thay đoạn DNS của ALB bằng DNS thật của bạn copy từ bên giao diện EC2 sang)*.
8. Bấm **Save**.

#### Tạo đường dẫn Real-time SSE Stream
Công nghệ Server-Sent Events (SSE) yêu cầu API Gateway phải mở luồng kết nối liên tục thay vì gom bộ đệm (buffer) trả về 1 lần.
1. Bấm vào thư mục gốc `/`, sau đó bấm **Create resource**. Tên: `api` > Create.
2. Bấm vào `/api`, bấm **Create resource**. Tên: `chat` > Create.
3. Bấm vào `/api/chat`, bấm **Create resource**. Tên: `stream` > Create.
4. Bấm vào `/api/chat/stream`, chọn **Create method**.
5. **Method type:** `GET`.
6. **Integration type:** Chọn **VPC Link**.
7. Bật **Use proxy integration**.
8. **Endpoint URL:** `http://internal-[DNS_ALB_CỦA_BẠN]/api/chat/stream`.
9. Bấm **Create**.

> [!IMPORTANT]
> Để streaming hoạt động, bạn phải vào phần **Integration Request** của phương thức `GET /api/chat/stream`, và đổi thông số **Response Transfer Mode** từ Buffered sang **Stream**.

#### Tạo đường dẫn phụ cho Chat (Sửa lỗi 404 Not Found)
Bởi vì chúng ta vừa tạo một đường dẫn cụ thể cho `/api/chat/stream`, API Gateway đã tự động sinh ra thư mục cha `/api/chat`. Điều này vô tình làm cho đường dẫn catch-all `/{proxy+}` gốc bị vô hiệu hóa đối với các request bắt đầu bằng `/api/chat/...` (ví dụ `/api/chat/message` hay `/api/chat/session`), dẫn đến lỗi 404 Not Found.
1. Bấm vào thư mục `/api/chat` và chọn **Create resource**.
2. Bật **Proxy resource**, để nguyên Resource path là `/{proxy+}`. Bấm **Create resource**.
3. Chọn phương thức `ANY` nằm dưới `/api/chat/{proxy+}`.
4. **Integration type:** Chọn **VPC Link**.
5. Bật **Use proxy integration**.
6. **VPC Link:** Chọn `pedix-vpclink`.
7. **Endpoint URL:** `http://internal-[DNS_ALB_CỦA_BẠN]/api/chat/{proxy}`.
8. Bấm **Save**.

#### Triển khai API (Deploy)
1. Bấm nút màu cam **Deploy API**.
2. Chọn **New stage**, đặt tên là `prod`, và bấm Deploy.
3. AWS sẽ cấp cho bạn một đường dẫn Invoke URL (VD: `https://96hnl890q4.execute-api.../prod`). 
4. Kiểm thử bằng cách dán `https://[URL_CỦA_BẠN]/prod/api/health` lên trình duyệt. Nếu hiện ra `{"status":"ok","service":"pedix-backend"}`, xin chúc mừng! Cây cầu bảo mật của bạn đã thông toàn tuyến!
