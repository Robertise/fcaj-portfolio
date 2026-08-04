---
title: "Tuần 4: Phát triển Bản Đề xuất & Mô hình Hệ thống"
date: 2026-05-11
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

### Tóm tắt Tuần 4
Hoàn thiện bản đề xuất dự án cho bootcamp. Chốt các sơ đồ kiến trúc và định nghĩa rõ ràng vòng lặp suy luận 5 giai đoạn cho trợ lý Pedix.

### Nhật ký Làm việc & Ghi chú Kỹ thuật
| Ngày | Mô tả Công việc | Thời lượng | Trạng thái | Ghi chú Kỹ thuật |
| :--- | :--- | :--- | :--- | :--- |
| Thứ Hai | Viết tài liệu Đề xuất Phần 1 | 7.5h | Hoàn thành | Viết tóm tắt dự án và bài toán. Nhấn mạnh sự nguy hiểm của các ứng dụng kiểm tra triệu chứng chung chung đối với trẻ em. |
| Thứ Ba | Viết tài liệu Đề xuất Phần 2 | 7.5h | Hoàn thành | Xác định kết quả kỳ vọng, cột mốc và dự toán ngân sách, đảm bảo tuân thủ nghiêm ngặt giới hạn AWS Free Tier. |
| Thứ Tư | Thiết kế Lọc An toàn Stage 0 | 7.5h | Hoàn thành | Thiết kế bộ lọc regex/keyword tất định (<10ms) để cờ báo ngay các triệu chứng nguy kịch (tím tái, li bì) trước khi gọi LLM. |
| Thứ Năm | Trực quan hóa Sơ đồ Kiến trúc | 8.0h | Hoàn thành | Vẽ luồng request từ CloudFront -> API Gateway -> VPC Link -> ALB -> EC2 trên Draw.io một cách chi tiết. |
| Thứ Sáu | Peer Review & Cải tiến Logic | 8.0h | Hoàn thành | Thảo luận đề xuất với nhóm. Tinh chỉnh logic phân loại ESI v4 cho phù hợp hơn với các đặc thù lâm sàng nhi khoa. |

### Kết quả Kỹ thuật Đạt được
- **Bản Đề xuất Chính thức:** Hoàn thành và chốt bản đề xuất dự án.
- **Bản vẽ Kỹ thuật:** Sơ đồ kiến trúc rõ ràng, sẵn sàng để triển khai.

> [!TIP]
> **Kỷ luật Kỹ thuật:** Toàn bộ mã nguồn và cấu hình AWS đều được kiểm thử gắt gao ở local trước khi đẩy lên cloud. Mọi sự cố phát sinh trong quá trình tích hợp (ví dụ: lỗi cấp quyền IAM, xung đột package) đều được ghi chú lại để hoàn thiện các quy trình CI/CD sau này.
