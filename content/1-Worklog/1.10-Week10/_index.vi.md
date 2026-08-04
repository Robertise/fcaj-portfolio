---
title: "Tuần 10: Tái định vị & Module Phân tích"
date: 2026-05-11
weight: 10
chapter: false
pre: " <b> 1.10. </b> "
---

### Tóm tắt Tuần 10
Chính thức đổi tên dự án thành 'Pedix'. Xây dựng dashboard phân tích cho admin sử dụng kỹ thuật truy vấn DynamoDB song song, và triển khai Phân quyền truy cập (RBAC) nghiêm ngặt qua Cognito.

### Nhật ký Làm việc & Ghi chú Kỹ thuật
| Ngày | Mô tả Công việc | Thời lượng | Trạng thái | Ghi chú Kỹ thuật |
| :--- | :--- | :--- | :--- | :--- |
| Thứ Hai | Đổi Tên Mã nguồn | 6.5h | Hoàn thành | Đổi toàn bộ tên package, biến môi trường và tài liệu từ PediCompass thành Pedix để đồng bộ thương hiệu. |
| Thứ Ba | Phân quyền RBAC Cognito | 8.0h | Hoàn thành | Tạo group `pedix-users` & `pedix-admins`. Viết Dependency `get_admin_user` trong FastAPI chặn JWT token sai. |
| Thứ Tư | Giao diện Analytics Dashboard | 7.5h | Hoàn thành | Dựng các view frontend cho trang quản trị, bao gồm các thẻ chỉ số (metric) và bảng log phiên người dùng. |
| Thứ Năm | Truy vấn Phân tích Song song | 7.5h | Hoàn thành | Dùng `asyncio.gather` của Python query đồng thời các bảng DynamoDB, giảm 60% thời gian load dashboard. |
| Thứ Sáu | Tính năng Pre-Visit Checklist | 7.5h | Hoàn thành | Dựng UI xử lý Markdown, tự động bóc tách và làm nổi bật phần 'Danh sách cần mang theo' từ Stage 5. |

### Kết quả Kỹ thuật Đạt được
- **Kiểm soát Quản trị:** Giao diện quản trị bảo mật 100% bằng RBAC.
- **Hiệu suất:** Tối ưu hóa mạnh mẽ khâu tổng hợp dữ liệu DynamoDB cho biểu đồ phân tích.

> [!TIP]
> **Kỷ luật Kỹ thuật:** Toàn bộ mã nguồn và cấu hình AWS đều được kiểm thử gắt gao ở local trước khi đẩy lên cloud. Mọi sự cố phát sinh trong quá trình tích hợp (ví dụ: lỗi cấp quyền IAM, xung đột package) đều được ghi chú lại để hoàn thiện các quy trình CI/CD sau này.
