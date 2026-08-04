---
title: "Tuần 7: Tối ưu Pipeline & Luồng Xác thực"
date: 2026-05-11
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---

### Tóm tắt Tuần 7
Nâng cao độ chính xác của pipeline truy xuất Qdrant bằng cách áp dụng bộ lọc payload metadata. Bắt đầu xây dựng bộ khung tích hợp Amazon Cognito để xác thực người dùng bảo mật.

### Nhật ký Làm việc & Ghi chú Kỹ thuật
| Ngày | Mô tả Công việc | Thời lượng | Trạng thái | Ghi chú Kỹ thuật |
| :--- | :--- | :--- | :--- | :--- |
| Thứ Hai | Lọc Qdrant Phân tầng Tuổi | 7.5h | Hoàn thành | Tạo KEYWORD payload index. LLM (Stage 1) trích xuất tuổi, Stage 2 lọc vector chính xác theo payload này. |
| Thứ Ba | Cải tiến Logic Stage 1 & 2 | 7.5h | Hoàn thành | Tinh chỉnh prompt để LLM luôn trả về đúng định dạng payload key ('newborn', 'toddler') để match với Qdrant. |
| Thứ Tư | Cập nhật Trình đọc Tài liệu | 6.5h | Hoàn thành | Nâng cấp script nạp dữ liệu dùng PyMuPDF để giữ nguyên format bảng biểu, cải thiện lớn chất lượng vector. |
| Thứ Năm | Giả lập Xác thực Cognito | 8.0h | Hoàn thành | Nghiên cứu luồng JWT của AWS Cognito. Tạo mock JWT validator trong middleware FastAPI để chuẩn bị tích hợp thật. |
| Thứ Sáu | Truyền Context Hồ sơ Bệnh nhi | 8.5h | Hoàn thành | Sửa endpoint chat để tự động chèn dữ liệu hồ sơ trẻ (cân nặng, bệnh nền) vào system prompt của Bedrock. |

### Kết quả Kỹ thuật Đạt được
- **Độ chính xác Truy xuất:** Xóa bỏ tình trạng LLM hallucinate (đưa ra lời khuyên của trẻ lớn cho trẻ sơ sinh).
- **Nhận diện Ngữ cảnh:** Chatbot tự động nắm bắt tiểu sử bệnh lý của người dùng.

> [!TIP]
> **Kỷ luật Kỹ thuật:** Toàn bộ mã nguồn và cấu hình AWS đều được kiểm thử gắt gao ở local trước khi đẩy lên cloud. Mọi sự cố phát sinh trong quá trình tích hợp (ví dụ: lỗi cấp quyền IAM, xung đột package) đều được ghi chú lại để hoàn thiện các quy trình CI/CD sau này.
