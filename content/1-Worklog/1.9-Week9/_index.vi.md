---
title: "Tuần 9: SSE Streaming & Suy luận Phân loại ESI v4"
date: 2026-05-11
weight: 9
chapter: false
pre: " <b> 1.9. </b> "
---

### Tóm tắt Tuần 9
Tuần quan trọng tập trung vào UX thời gian thực và logic lâm sàng phức tạp. Triển khai Server-Sent Events (SSE) để stream 5 giai đoạn suy luận lên UI, đồng thời mã hóa bộ quy tắc phân loại cấp cứu ESI v4.

### Nhật ký Làm việc & Ghi chú Kỹ thuật
| Ngày | Mô tả Công việc | Thời lượng | Trạng thái | Ghi chú Kỹ thuật |
| :--- | :--- | :--- | :--- | :--- |
| Thứ Hai | Triển khai Luồng SSE | 8.0h | Hoàn thành | Chuyển đổi endpoint `/api/chat` sang `StreamingResponse`, trả về từng cục JSON lũy tiến. |
| Thứ Ba | Hiển thị Vết suy luận Real-time | 7.5h | Hoàn thành | Lập trình UI state để mở khóa dần các bước suy luận nội bộ của AI ngay khi mỗi stage kết thúc. |
| Thứ Tư | Cơ chế Heartbeat cho SSE | 6.5h | Hoàn thành | Thêm frame heartbeat rỗng mỗi 5 giây. Ngăn CloudFront và ALB ngắt kết nối do idle khi LLM sinh text quá lâu. |
| Thứ Năm | Tích hợp Logic ESI v4 | 7.5h | Hoàn thành | Nhúng bộ quy tắc Emergency Severity Index v4 vào prompt để ép AI phân loại chính xác các mức độ cấp cứu (1-5). |
| Thứ Sáu | Xác thực Intent Trò chuyện | 7.0h | Hoàn thành | Bổ sung màng lọc Bedrock pre-flight chặn các câu hỏi phi y tế, từ chối khéo léo để tiết kiệm token API. |

### Kết quả Kỹ thuật Đạt được
- **UX Thời gian thực:** Phụ huynh thấy ngay 'quá trình suy nghĩ' của AI thay vì phải chờ 15s để ra kết quả cuối.
- **An toàn Lâm sàng:** Tuân thủ chặtặt chẽ việc phân loại cấp cứu chuẩn ESI v4.

> [!TIP]
> **Kỷ luật Kỹ thuật:** Toàn bộ mã nguồn và cấu hình AWS đều được kiểm thử gắt gao ở local trước khi đẩy lên cloud. Mọi sự cố phát sinh trong quá trình tích hợp (ví dụ: lỗi cấp quyền IAM, xung đột package) đều được ghi chú lại để hoàn thiện các quy trình CI/CD sau này.
