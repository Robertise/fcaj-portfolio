---
title: "Bước 4: Phân phối Frontend, Kiểm thử & Clean-up"
date: 2024-01-01
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

Trong bước cuối cùng này, chúng ta phân phối ứng dụng **React (Vite)** lên **Amazon S3** & **CloudFront (OAC)**, thực hiện kiểm thử E2E luồng SSE streaming, cấu hình theo dõi chi phí và dọn dẹp tài nguyên.

---

### 1. Build Frontend & Đưa lên CloudFront
1. Đóng gói mã nguồn React tại local:
   ```bash
   npm run build
   ```
2. Khởi tạo Amazon S3 Bucket `pedix-frontend-static-singapore` và đồng bộ tài nguyên:
   ```bash
   aws s3 sync dist/ s3://pedix-frontend-static-singapore --delete
   ```
3. Tạo **CloudFront Distribution**:
   - **Nguồn (Origin):** S3 Bucket gán Origin Access Control (OAC).
   - **Chính sách xem:** Chuyển hướng HTTP sang HTTPS.
   - **Cấu hình lỗi SPA:** Trỏ 403/404 về `/index.html`.

---

### 2. Kiểm thử E2E & Xác minh SSE Streaming
1. Truy cập đường dẫn domain CloudFront trên trình duyệt.
2. Đăng ký và đăng nhập qua Cognito.
3. Nhập testcase kiểm thử: *"Trẻ 3 tháng tuổi sốt 38.5°C kèm ho nhẹ."*
4. Kiểm tra vết suy luận SSE hiển thị thời gian thực trên giao diện:
   - **Stage 0:** Bộ lọc an toàn định tính thông qua (<10ms)
   - **Stage 1:** Tuổi được ánh xạ vào `young_infant` (90 ngày)
   - **Stage 2:** Tìm kiếm vector Qdrant lọc `age_group: young_infant`
   - **Stage 3:** Phân loại ESI v4: Mức 2 (Khẩn cấp — cần đưa trẻ đi khám nhi khoa)
   - **Stage 4:** Vòng lặp Reflection hoàn thành
   - **Stage 5:** Trả về hướng dẫn chăm sóc kèm nguồn trích dẫn WHO.

---

### 3. Giám sát Chi phí AWS Budgets & CloudWatch
- **AWS Budgets:** Đã tạo cảnh báo gửi email khi chi phí vượt ngưỡng **$50.00**.
- **CloudWatch Logs:** Cấu hình thời gian lưu log 14 ngày cho `/aws/ec2/pedix-fastapi`.

---

### 4. Hướng dẫn Hủy bỏ Tài nguyên (Clean-up)
Để xóa bỏ các tài nguyên tránh phát sinh chi phí sau khi hoàn thành:
1. **S3 Bucket:** Xóa sạch đối tượng và xóa bucket `pedix-frontend-static-singapore`.
2. **CloudFront:** Vô hiệu hóa và xóa distribution.
3. **API Gateway & VPC Link:** Xóa REST API `Pedix-API` và VPC Link V2.
4. **Internal ALB & Target Group:** Xóa ALB và `pedix-backend-tg`.
5. **EC2 Instance:** Terminate máy chủ `pedix-backend-server`.
6. **DynamoDB Tables:** Xóa 4 bảng `pedix_sessions`, `pedix_profiles`, `pedix_analytics_log`, `pedix_documents`.
7. **Cognito User Pool:** Xóa `pedix-user-pool`.