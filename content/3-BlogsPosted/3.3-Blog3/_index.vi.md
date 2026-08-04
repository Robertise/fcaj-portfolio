---
title: "Blog 3"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.3. </b> "
---
{{% notice warning %}}
⚠️ **Lưu ý:** Các thông tin dưới đây chỉ nhằm mục đích tham khảo, vui lòng **không sao chép nguyên văn** cho bài báo cáo của bạn kể cả warning này.
{{% /notice %}}

# BẢO MẬT BACKEND NỘI BỘ VỚI API GATEWAY VPC LINK V2 & INTERNAL ALB

### Tổng quan Kiến trúc
Trong nguyên tắc bảo mật ứng dụng đám mây (DevSecOps), máy chủ backend EC2 không bao giờ được phép mở port HTTP trực tiếp ra internet công cộng. Bài viết này mô tả chi tiết thiết kế mạng Zero-Trust được triển khai cho dự án **Pedix**, sử dụng **Amazon API Gateway REST API**, **VPC Link V2** và **Internal Application Load Balancer (ALB)**.

---

### Luồng Dữ liệu & Triển khai Chi tiết

#### 1. Cổng Tiếp Nhận Công Cộng & Giới Hạn Tốc Độ
Yêu cầu từ người dùng gửi tới Amazon API Gateway qua HTTPS. API Gateway thi hành chính sách CORS, giới hạn số lượng truy vấn (throttling 100 req/s) và kiểm tra JWT token từ Cognito User Pool.

#### 2. Kênh Truyền Riêng Tư qua VPC Link V2
Thay vì truyền qua IP công cộng, API Gateway kết nối trực tiếp vào private subnet của VPC nhờ **VPC Link V2**. Cơ chế này tạo các giao diện mạng ảo ENI (Elastic Network Interface) đặt trực tiếp trong VPC để truyền tải request một cách an toàn tuyệt đối.

#### 3. Cân Bằng Tải Nội Bộ & Security Group Phân Tầng
VPC Link chuyển hướng traffic tới **Internal Application Load Balancer (ALB)**. Security Group của ALB được cấu hình *chỉ chấp nhận* lưu lượng đến từ CIDR subnet của VPC Link. Tương tự, Security Group của EC2 backend *chỉ cho phép* nhận HTTP port 8000 từ duy nhất ALB Security Group.

```
[Khách hàng] ---> (CloudFront / HTTPS) ---> [API Gateway] 
                                                  |
                                          (VPC Link V2)
                                                  |
                                       [Internal ALB (Port 80)]
                                                  |
                                       [EC2 FastAPI (Port 8000)]
```

#### 4. Duy Trì Kết Nối SSE Streaming (Keep-Alive)
Do quá trình suy luận lâm sàng trên Amazon Bedrock có thể kéo dài 5-15 giây, các timeout mặc định (29 giây trên API Gateway) có thể gây ngắt kết nối giữa chừng. Backend FastAPI được tích hợp cơ chế phát heartbeat định kỳ mỗi 5 giây (`: ping\n\n`), giữ cho luồng VPC Link luôn hoạt động thông suốt để giao diện React hiển thị vết suy luận thời gian thực.