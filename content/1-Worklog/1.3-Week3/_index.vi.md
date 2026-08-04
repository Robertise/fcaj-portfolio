---
title: "Tuần 3: Tổng hợp Tri thức & Phác thảo Kiến trúc"
date: 2026-05-11
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---

### Tóm tắt Tuần 3
Tập trung vào khâu chuẩn bị dữ liệu và thiết kế hệ thống. Trích xuất và làm sạch hàng nghìn dòng hướng dẫn lâm sàng từ file PDF của WHO bằng công cụ MinerU. Phác thảo sơ đồ kiến trúc đám mây sơ bộ.

### Nhật ký Làm việc & Ghi chú Kỹ thuật
| Ngày | Mô tả Công việc | Thời lượng | Trạng thái | Ghi chú Kỹ thuật |
| :--- | :--- | :--- | :--- | :--- |
| Thứ Hai | Trích xuất Hướng dẫn WHO | 8.5h | Hoàn thành | Trích xuất text từ 'Pocket book of hospital care for children' bằng MinerU. Xử lý các lỗi nhận diện OCR phức tạp. |
| Thứ Ba | Cấu trúc và Làm sạch Dữ liệu | 6.5h | Hoàn thành | Định dạng lại thủ công các bảng markdown, danh sách và tiêu đề để đảm bảo chất lượng văn bản cao nhất cho Vector DB. |
| Thứ Tư | Chiến lược Semantic Chunking | 7.0h | Hoàn thành | Nghiên cứu Contextual Retrieval. Thiết kế bộ chia chunk dựa trên Markdown (300-500 token), overlap 50 token. |
| Thứ Năm | Phác thảo Kiến trúc Hệ thống | 8.0h | Hoàn thành | Lập bản đồ dịch vụ AWS: API Gateway, EC2 (Backend/Qdrant), Bedrock, và DynamoDB (Tối ưu giới hạn Free Tier). |
| Thứ Sáu | Thiết kế Mạng Zero-Trust | 7.5h | Hoàn thành | Lên kế hoạch Security Group Chaining để đảm bảo EC2 backend bị cô lập hoàn toàn khỏi truy cập internet trực tiếp. |

### Kết quả Kỹ thuật Đạt được
- **Chuẩn bị Dữ liệu:** Chuyển đổi 3 sách PDF y khoa phức tạp thành file Markdown sạch, sẵn sàng chunking.
- **Tầm nhìn Kiến trúc:** Thiết lập bản thiết kế rõ ràng cho một hệ thống AWS an toàn, chi phí thấp.

> [!TIP]
> **Kỷ luật Kỹ thuật:** Toàn bộ mã nguồn và cấu hình AWS đều được kiểm thử gắt gao ở local trước khi đẩy lên cloud. Mọi sự cố phát sinh trong quá trình tích hợp (ví dụ: lỗi cấp quyền IAM, xung đột package) đều được ghi chú lại để hoàn thiện các quy trình CI/CD sau này.
