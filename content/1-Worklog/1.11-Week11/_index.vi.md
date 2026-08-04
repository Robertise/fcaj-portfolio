---
title: "Tuần 11: Phân quyền Bảo mật & Cấu hình Mô hình AI"
date: 2026-05-11
weight: 11
chapter: false
pre: " <b> 1.11. </b> "
---

### Tóm tắt Tuần 11
Chốt các lớp bảo mật kiến trúc đám mây. Cấu hình IAM Policy chi tiết và đánh giá giới hạn băng thông mô hình Bedrock, quyết định chọn Claude Haiku để cân bằng chi phí và hiệu năng.

### Nhật ký Làm việc & Ghi chú Kỹ thuật
| Ngày | Mô tả Công việc | Thời lượng | Trạng thái | Ghi chú Kỹ thuật |
| :--- | :--- | :--- | :--- | :--- |
| Thứ Hai | Kiểm thử E2E Local Phần 1 | 8.5h | Hoàn thành | Chạy bài test tích hợp quy mô lớn. Kiểm tra mọi luồng UI và các vết suy luận RAG (Agentic Reasoning traces). |
| Thứ Ba | Kiểm thử E2E Local Phần 2 | 7.5h | Hoàn thành | Kiểm tra các kết nối DynamoDB mock và các luồng xác thực Cognito trước khi tiến hành đẩy lên Cloud. |
| Thứ Tư | Phân quyền IAM Tối thiểu | 7.0h | Hoàn thành | Tạo `Pedix-EC2-Role` khóa quyền DynamoDB. Xóa bỏ hoàn toàn hardcode credential trên server EC2. |
| Thứ Năm | Rà soát Môi trường AWS Cloud | 8.5h | Hoàn thành | Kiểm tra các hạn mức (quotas) của region `ap-southeast-1` để đảm bảo đủ tài nguyên triển khai production. |
| Thứ Sáu | Chuyển đổi Mô hình Bedrock | 8.5h | Hoàn thành | Chuyển từ Claude 3.5 Sonnet sang Claude 3 Haiku để vượt qua rào cản rate limit cực gắt (ThrottlingException) của Free Tier. |

### Kết quả Kỹ thuật Đạt được
- **Tuân thủ Bảo mật:** Đảm bảo 100% nguyên tắc quyền hạn tối thiểu IAM.
- **Độ ổn định:** Vượt qua rào cản giới hạn request của AI trên môi trường Free Tier.

> [!TIP]
> **Kỷ luật Kỹ thuật:** Toàn bộ mã nguồn và cấu hình AWS đều được kiểm thử gắt gao ở local trước khi đẩy lên cloud. Mọi sự cố phát sinh trong quá trình tích hợp (ví dụ: lỗi cấp quyền IAM, xung đột package) đều được ghi chú lại để hoàn thiện các quy trình CI/CD sau này.
