---
title: "Yêu cầu tiên quyết"
date: 2024-01-01 
weight: 2 
chapter: false
pre: " <b> 5.2. </b> "
---

Trước khi bắt đầu triển khai **Pedix** trên AWS, hãy đảm bảo bạn đã chuẩn bị đầy đủ các tài khoản, dịch vụ và công cụ bên dưới.

---

### 1. Tài khoản AWS & Region
- **AWS Region:** Chọn Region `ap-southeast-1` (Singapore) để đảm bảo độ trễ thấp và có sẵn dịch vụ Amazon Bedrock.
- **Quyền QTV IAM:** Cần quyền quản trị để khởi tạo IAM Roles, bảng DynamoDB, VPC Link và CloudFront distribution.

---

### 2. Kích hoạt Mô hình Amazon Bedrock
1. Truy cập **Amazon Bedrock Console** -> **Model Access**.
2. Yêu cầu cấp quyền truy cập các model:
   - **Anthropic Claude 3.5 Sonnet** (cho suy luận lâm sàng & reflection)
   - **Anthropic Claude 3 Haiku** (cho phân tích truy vấn & safety screen)

---

### 3. Cấu hình IAM Role (`Pedix-EC2-Role`)
Tạo IAM Role gán cho máy chủ EC2 với các chính sách (Managed Policies):
- `AmazonBedrockFullAccess`
- `AmazonDynamoDBFullAccess`
- `CloudWatchLogsFullAccess`

---

### 4. Công cụ Lập trình Cần thiết
- **AWS CLI v2:** Đã cài đặt và cấu hình credentials (`aws configure`).
- **Python 3.11+:** Dùng để chạy script test và nạp tri thức y khoa.
- **Docker Desktop:** Dùng để kiểm thử container Qdrant tại máy local.