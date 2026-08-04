---
title: "Bước 2: Mạng Riêng & API Gateway"
date: 2024-01-01
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

Trong bước này, chúng ta thiết lập hạ tầng mạng riêng tư Zero-Trust kết nối **Amazon API Gateway** với máy chủ **EC2 Backend** thông qua **VPC Link V2** và **Internal Application Load Balancer (ALB)**.

---

### 1. Khởi tạo Internal ALB (Application Load Balancer Nội bộ)
1. Truy cập **EC2 Console** -> **Load Balancers** -> **Create Load Balancer**.
2. Chọn loại **Application Load Balancer**:
   - **Chế độ (Scheme):** `Internal` (Chỉ chấp nhận IP riêng tư trong VPC)
   - **Loại IP:** IPv4
   - **Bộ lắng nghe (Listener):** HTTP Port 80
3. Khởi tạo Target Group `pedix-backend-tg`:
   - **Loại Target:** Instance
   - **Giao thức/Port:** HTTP / 8000
   - **Đường dẫn Health Check:** `/health`
   - Đăng ký `pedix-backend-server` làm target.

---

### 2. Cấu hình VPC Link V2
1. Truy cập **API Gateway Console** -> **VPC Links** -> **Create VPC Link**.
2. Chọn **VPC Link for HTTP / REST APIs (V2)**.
3. Chọn VPC và subnets nội bộ để khởi tạo card mạng ảo ENI.

---

### 3. Tạo REST API Proxy & Hỗ trợ SSE Streaming
1. Khởi tạo REST API với tên `Pedix-API`.
2. Tạo proxy resource `{proxy+}` với phương thức `ANY`:
   - **Loại tích hợp:** VPC Link
   - **VPC Link:** Chọn VPC Link V2 vừa tạo
   - **Endpoint URL:** `http://<internal-alb-dns>/{proxy}`
3. Bật cấu hình CORS và deploy stage `prod`.
