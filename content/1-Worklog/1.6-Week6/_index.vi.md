---
title: "Tuần 6: Thiết kế Giao diện & Điều phối Dịch vụ"
date: 2026-05-11
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

### Tóm tắt Tuần 6
Chuyển trọng tâm sang trải nghiệm người dùng frontend. Thiết kế ứng dụng React chuẩn mobile-first bằng TailwindCSS và sử dụng Docker Compose để điều phối các dịch vụ backend local.

### Nhật ký Làm việc & Ghi chú Kỹ thuật
| Ngày | Mô tả Công việc | Thời lượng | Trạng thái | Ghi chú Kỹ thuật |
| :--- | :--- | :--- | :--- | :--- |
| Thứ Hai | Khởi tạo Khung React UI | 7.5h | Hoàn thành | Tạo React SPA bằng Vite. Cấu hình TailwindCSS và React Router cho các trang Home, Chat, Profile. |
| Thứ Ba | Xây dựng Giao diện Chatbot (Phần 1) | 7.0h | Hoàn thành | Thiết kế sidebar dạng collapsible để hiển thị 'Vết suy luận' (Reasoning Trace) cho phép người dùng kiểm tra AI. |
| Thứ Tư | Xây dựng Giao diện Chatbot (Phần 2) | 8.0h | Hoàn thành | Lập trình tính năng bong bóng chat tự cuộn, hiệu ứng typing indicator và thư viện render Markdown. |
| Thứ Năm | Điều phối Dịch vụ Local | 8.0h | Hoàn thành | Viết `docker-compose.yml` map port FastAPI (8000) và Qdrant (6333). Đảm bảo mapping volume bền vững. |
| Thứ Sáu | Kết nối Frontend-Backend | 8.5h | Hoàn thành | Gắn Axios client của React vào các endpoint của FastAPI. Xử lý triệt để CORS preflight `OPTIONS` requests. |

### Kết quả Kỹ thuật Đạt được
- **Giao diện Hoàn thiện:** Khung chat mượt mà, tối ưu hiển thị trên di động.
- **Điều phối Dịch vụ:** Khởi động toàn bộ local system chỉ với 1 lệnh Docker Compose.

> [!TIP]
> **Kỷ luật Kỹ thuật:** Toàn bộ mã nguồn và cấu hình AWS đều được kiểm thử gắt gao ở local trước khi đẩy lên cloud. Mọi sự cố phát sinh trong quá trình tích hợp (ví dụ: lỗi cấp quyền IAM, xung đột package) đều được ghi chú lại để hoàn thiện các quy trình CI/CD sau này.
