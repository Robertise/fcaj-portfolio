---
title: "Tuần 2: Nghiên cứu RAG & Xác định Phạm vi Nhi khoa"
date: 2026-05-11
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---

### Tóm tắt Tuần 2
Nghiên cứu sâu về mô hình Retrieval-Augmented Generation (RAG). Sau khi thảo luận với mentor, phạm vi dự án được thu hẹp nghiêm ngặt thành trợ lý y tế nhi khoa cho trẻ dưới 5 tuổi (Pedix).

### Nhật ký Làm việc & Ghi chú Kỹ thuật
| Ngày | Mô tả Công việc | Thời lượng | Trạng thái | Ghi chú Kỹ thuật |
| :--- | :--- | :--- | :--- | :--- |
| Thứ Hai | Nghiên cứu Framework Agentic RAG | 6.5h | Hoàn thành | Phân tích RAG Level 1-4. Thiết kế luồng suy luận đa giai đoạn minh bạch (Lập kế hoạch -> Truy xuất -> Suy ngẫm -> Trả lời). |
| Thứ Ba | Đánh giá Cơ sở dữ liệu Vector | 8.5h | Hoàn thành | So sánh Pinecone, OpenSearch, và Qdrant. Chọn Qdrant vì hỗ trợ Docker local và tốn ít tài nguyên RAM. |
| Thứ Tư | Phân loại dữ liệu y tế tổng quát | 7.5h | Hoàn thành | Tổng hợp dữ liệu y khoa tổng quát thành 14 danh mục lớn trước khi nhận ra phạm vi hiện tại là quá rộng. |
| Thứ Năm | Thảo luận chuyên môn với Mentor | 8.0h | Hoàn thành | Thảo luận với mentor. Chuyển hướng sang nhi khoa (dưới 5 tuổi) do có các ngưỡng lâm sàng khắt khe và thực tế. |
| Thứ Sáu | Xác định Phạm vi Nhi khoa chi tiết | 7.5h | Hoàn thành | Phác thảo các mốc tuổi cụ thể (sơ sinh, nhũ nhi, trẻ nhỏ) và ánh xạ sơ bộ các mức độ khẩn cấp y tế. |

### Kết quả Kỹ thuật Đạt được
- **Quyết định Kiến trúc:** Áp dụng mô hình Agentic RAG Level 4.
- **Chiến lược Dữ liệu:** Chốt phương án chỉ sử dụng tài liệu hướng dẫn nhi khoa chuẩn của WHO và NICE.

> [!TIP]
> **Kỷ luật Kỹ thuật:** Toàn bộ mã nguồn và cấu hình AWS đều được kiểm thử gắt gao ở local trước khi đẩy lên cloud. Mọi sự cố phát sinh trong quá trình tích hợp (ví dụ: lỗi cấp quyền IAM, xung đột package) đều được ghi chú lại để hoàn thiện các quy trình CI/CD sau này.
