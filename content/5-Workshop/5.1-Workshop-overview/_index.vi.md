---
title: "1. Tổng quan Workshop"
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

# Triển khai Hệ thống Agentic RAG Nhi khoa trên AWS

Workshop này sẽ hướng dẫn bạn chi tiết từng bước (step-by-step) để triển khai **Pedix** - một hệ thống Agentic Retrieval-Augmented Generation (RAG) cấp độ Production dành cho tư vấn y tế nhi khoa.

Kết thúc workshop, bạn sẽ tự tay xây dựng được một kiến trúc đám mây Zero-Trust bảo mật cao, kết hợp 12 dịch vụ AWS và được tối ưu hóa hoàn toàn cho môi trường Production thực tế.

---

### Sơ đồ Kiến trúc

![PediCompass AWS Cloud Architecture](/images/2-Proposal/pedix_architecture.png)

### Luồng Dữ liệu (Cô lập Mạng Zero-Trust)

Kiến trúc của chúng tôi tuân thủ nghiêm ngặt thiết kế **Zero-Trust Network Isolation**:
1. **Người dùng (Trình duyệt):** Kết nối tới frontend React lưu trữ trên **Amazon S3** và phân phối siêu tốc qua **Amazon CloudFront** CDN.
2. **API Requests:** Các lệnh gọi API đi tới **Amazon API Gateway**. Tại đây, hệ thống tự động xác thực token JWT của người dùng bằng **Amazon Cognito** Authorizer.
3. **Đường hầm VPC Link:** Các request hợp lệ sẽ được "đào hầm" an toàn đi thẳng vào Default VPC (`172.31.0.0/16`) thông qua **VPC Link V2**.
4. **Định tuyến Nội bộ:** Traffic chạm tới **Internal Application Load Balancer (ALB)**. ALB này KHÔNG có Public IP và hoàn toàn ẩn (private).
5. **EC2 Backend:** ALB chuyển tiếp request tới máy chủ **EC2** chạy FastAPI (Cổng 8000). Security Group được thiết lập chuỗi (chaining) để Cổng 8000 CHỈ nhận traffic từ ALB. Mọi truy cập trực tiếp từ internet bị chặn đứng 100%.
6. **Cơ sở dữ liệu Vector:** **Qdrant** chạy bằng Docker, bị trói buộc (bound) hoàn toàn vào `127.0.0.1:6333` (localhost loopback), đảm bảo không một ai trên mạng có thể quét thấy nó.
7. **Dịch vụ Đám mây:** EC2 backend giao tiếp hướng ra ngoài (outbound) với **Amazon Bedrock** (Claude Haiku) và **Amazon DynamoDB** để suy luận AI và lưu trữ.

> [!TIP]
> **Tại sao dùng Default VPC thay vì Private Subnet + NAT Gateway?**
> Kiến trúc tiêu chuẩn doanh nghiệp thường đặt backend vào Private Subnet nằm sau NAT Gateway. Tuy nhiên, NAT Gateway ngốn tới ~$32/tháng. Bằng cách triển khai trong Default VPC kết hợp Internal ALB và Security Group chaining (chặn mọi IP truy cập thẳng vào EC2 trừ ALB), chúng ta đạt được **độ cô lập Zero-Trust y hệt** trong khi loại bỏ được các chi phí không cần thiết.

---

### Bảng Chi phí Production

Kiến trúc này được thiết kế và tối ưu cho môi trường Production (cân bằng giữa bảo mật, hiệu năng và chi phí). Ước tính chi phí hàng tháng là khoảng **~$72.7/tháng**.

| Dịch vụ | Ước tính Chi phí / Tháng |
|---|---|
| EC2 t3.micro (Singapore) | ~$9.50 |
| Public IPv4 | $3.65 |
| EBS Storage (30 GiB gp3) | ~$3.00 |
| Application Load Balancer | ~$24.24 |
| API Gateway (VPC Link REST) | $18.25 |
| API Gateway Requests | ~$0.00 |
| **Amazon Bedrock (Claude Haiku)** | **~$14.00** |
| Amazon DynamoDB | <$0.10 |
| Amazon Cognito & CloudWatch | $0.00 |
| Data Transfer | ~$0.05 |
| **TỔNG CỘNG** | **~$72.7/tháng** |
