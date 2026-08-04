---
title: "Tuần 5: Nền tảng Backend & Tích hợp Qdrant"
date: 2026-05-11
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

### Tóm tắt Tuần 5
Bắt đầu giai đoạn lập trình cốt lõi. Khởi tạo backend FastAPI, cài đặt Qdrant qua Docker local và triển khai các prompt LLM nền tảng sử dụng Anthropic Claude Haiku.

### Nhật ký Làm việc & Ghi chú Kỹ thuật
| Ngày | Mô tả Công việc | Thời lượng | Trạng thái | Ghi chú Kỹ thuật |
| :--- | :--- | :--- | :--- | :--- |
| Thứ Hai | Cấu trúc Project FastAPI | 7.0h | Hoàn thành | Khởi tạo cấu trúc thư mục module hóa (api, core, services). Cấu hình CORS middleware và error handler toàn cục. |
| Thứ Ba | Database Mock Models | 8.5h | Hoàn thành | Định nghĩa Pydantic schema cho User Profiles, Sessions và Analytics. Chuẩn bị interface để tích hợp DynamoDB sau này. |
| Thứ Tư | Cài đặt Qdrant Vector DB (Docker) | 6.5h | Hoàn thành | Pull Docker image Qdrant v1.10.1. Khởi tạo collection `pedix_kb` với vector 384 chiều, khoảng cách cosine. |
| Thứ Năm | Cấu hình Local & Boto3 | 7.5h | Hoàn thành | Cấu hình AWS Profile ở local và thiết lập các biến môi trường để truy cập an toàn API của Amazon Bedrock. |
| Thứ Sáu | Kỹ thuật Prompt Bedrock | 8.0h | Hoàn thành | Viết system prompt bọc bằng thẻ XML cho Phân tích Truy vấn (Stage 1) và Suy luận Lâm sàng (Stage 3). |

### Kết quả Kỹ thuật Đạt được
- **Backend Hoạt động:** FastAPI server chạy ổn định ở local port 8000.
- **Kết nối AI:** Gọi thành công Amazon Bedrock Claude Haiku thông qua Boto3.

> [!TIP]
> **Kỷ luật Kỹ thuật:** Toàn bộ mã nguồn và cấu hình AWS đều được kiểm thử gắt gao ở local trước khi đẩy lên cloud. Mọi sự cố phát sinh trong quá trình tích hợp (ví dụ: lỗi cấp quyền IAM, xung đột package) đều được ghi chú lại để hoàn thiện các quy trình CI/CD sau này.
