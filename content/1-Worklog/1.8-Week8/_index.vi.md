---
title: "Tuần 8: Mở rộng Nạp dữ liệu & Tái cấu trúc DynamoDB"
date: 2026-05-11
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---

### Tóm tắt Tuần 8
Hoàn thiện pipeline nạp dữ liệu, nhúng thành công toàn bộ tài liệu hướng dẫn lâm sàng. Tái cấu trúc toàn bộ tầng database của backend để kết nối mượt mà với các bảng Amazon DynamoDB On-Demand.

### Nhật ký Làm việc & Ghi chú Kỹ thuật
| Ngày | Mô tả Công việc | Thời lượng | Trạng thái | Ghi chú Kỹ thuật |
| :--- | :--- | :--- | :--- | :--- |
| Thứ Hai | Script Nạp Dữ liệu Hàng loạt | 6.5h | Hoàn thành | Viết script Python dùng `sentence-transformers` (build CPU) để batch embed hàng loạt tài liệu WHO. |
| Thứ Ba | Xử lý Embedding Qdrant | 8.0h | Hoàn thành | Thực thi kịch bản batch ingestion, ánh xạ toàn bộ 14 danh mục lâm sàng vào collection `pedix_kb`. |
| Thứ Tư | Tái cấu trúc DynamoDB Phần 1 | 8.5h | Hoàn thành | Thay thế mock storage bằng Boto3. Áp dụng các lệnh `Query` và `PutItem` tối ưu cho cơ sở dữ liệu thực. |
| Thứ Năm | Xử lý DB Bất đồng bộ | 7.5h | Hoàn thành | Đưa các lệnh Boto3 (vốn là blocking) vào `run_in_threadpool` của FastAPI để tránh nghẽn event loop ASGI. |
| Thứ Sáu | Thiết kế Khóa DynamoDB | 8.0h | Hoàn thành | Thiết kế Khóa chính Phức hợp (Partition Key: `user_id`, Sort Key: `profile_id`) để loại bỏ hoàn toàn lệnh `Scan`. |

### Kết quả Kỹ thuật Đạt được
- **Sẵn sàng Database:** Backend tích hợp chuẩn pattern DynamoDB cấp độ production.
- **Tri thức Đã nhúng:** Hơn 10.000 dòng hướng dẫn lâm sàng đã được vector hóa thành công.

> [!TIP]
> **Kỷ luật Kỹ thuật:** Toàn bộ mã nguồn và cấu hình AWS đều được kiểm thử gắt gao ở local trước khi đẩy lên cloud. Mọi sự cố phát sinh trong quá trình tích hợp (ví dụ: lỗi cấp quyền IAM, xung đột package) đều được ghi chú lại để hoàn thiện các quy trình CI/CD sau này.
