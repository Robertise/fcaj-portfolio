---
title: "Tuần 12: Triển khai AWS Zero-Trust & Nghiệm thu"
date: 2026-05-11
weight: 12
chapter: false
pre: " <b> 1.12. </b> "
---

### Tóm tắt Tuần 12
Thực hiện triển khai production lên server thật. Điều phối mạng Zero-Trust kết hợp API Gateway VPC Link, Internal ALB và Cognito. Hệ thống chính thức live trên domain CloudFront.

### Nhật ký Làm việc & Ghi chú Kỹ thuật
| Ngày | Mô tả Công việc | Thời lượng | Trạng thái | Ghi chú Kỹ thuật |
| :--- | :--- | :--- | :--- | :--- |
| Thứ Hai | Triển khai EC2, ALB & Qdrant | 6.5h | Hoàn thành | Tạo EC2 `t3.micro` có 2GB Swap. Cấu hình Internal ALB. Chạy Qdrant & FastAPI backend. Các bảng DynamoDB tự động tạo. |
| Thứ Ba | Cấu hình API GW & VPC Link | 8.0h | Hoàn thành | Thiết lập REST API Gateway bảo mật bằng Cognito Authorizer. Dẫn traffic ngầm vào ALB thông qua VPC Link V2 (`fzvy02`). |
| Thứ Tư | Triển khai S3 & CDN CloudFront | 7.5h | Hoàn thành | Build React SPA. Tải file tĩnh lên S3 nội bộ. Tạo distribution CloudFront dùng Origin Access Control (OAC) và SPA fallback. |
| Thứ Năm | Kiểm thử Live End-to-End | 8.0h | Hoàn thành | Chạy test toàn diện trên domain CloudFront public. Xác minh các API health check và độ ổn định của SSE streaming. |
| Thứ Sáu | Nghiệm thu Dự án & Slides | 7.5h | Hoàn thành | Tổng hợp toàn bộ tài liệu kỹ thuật, xác minh dự án đáp ứng đủ yêu cầu và chốt các slide thuyết trình cuối cùng. |

### Kết quả Kỹ thuật Đạt được
- **Live Production:** Hệ thống triển khai thành công, truy cập mượt mà qua CloudFront.
- **Zero-Trust Đạt chuẩn:** Backend EC2 cô lập 100% khỏi internet công cộng, chỉ nhận luồng dữ liệu sạch đã qua trạm kiểm duyệt API Gateway VPC Link.

> [!TIP]
> **Kỷ luật Kỹ thuật:** Toàn bộ mã nguồn và cấu hình AWS đều được kiểm thử gắt gao ở local trước khi đẩy lên cloud. Mọi sự cố phát sinh trong quá trình tích hợp (ví dụ: lỗi cấp quyền IAM, xung đột package) đều được ghi chú lại để hoàn thiện các quy trình CI/CD sau này.
