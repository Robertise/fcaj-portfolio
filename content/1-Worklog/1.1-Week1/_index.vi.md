---
title: "Tuần 1: Nền tảng & Khởi tạo AWS Cloud"
date: 2026-05-11
weight: 1
chapter: false
pre: " <b> 1.1. </b> "
---

### Tóm tắt Tuần 1
Tuần này tập trung vào việc trau dồi kiến thức nền tảng AWS Cloud thông qua AWS Educate, thiết lập tài khoản chính với cảnh báo thanh toán và lên ý tưởng sơ bộ cho dự án. Tôi đã tìm hiểu về các dịch vụ điện toán, lưu trữ và mạng cốt lõi.

### Nhật ký Làm việc & Ghi chú Kỹ thuật
| Ngày | Mô tả Công việc | Thời lượng | Trạng thái | Ghi chú Kỹ thuật |
| :--- | :--- | :--- | :--- | :--- |
| Thứ Hai | Tạo tài khoản AWS & Cấu hình Billing | 7.0h | Hoàn thành | Thiết lập tài khoản AWS Free Tier. Cài đặt cảnh báo AWS Budgets ở mức $10 để kiểm soát chi phí. |
| Thứ Ba | AWS Educate: Các Dịch vụ Lõi | 7.5h | Hoàn thành | Hoàn thành các module EC2, S3, VPC và IAM. Nghiên cứu nguyên tắc quyền hạn tối thiểu (Least Privilege). |
| Thứ Tư | Nghiên cứu Thị trường Generative AI y tế | 8.0h | Hoàn thành | Phân tích xu hướng cộng đồng về ứng dụng AI trong y tế và xem xét các bộ dữ liệu tiềm năng. |
| Thứ Năm | Thiết lập Git Repository & Worklog | 8.0h | Hoàn thành | Khởi tạo local Git repository và các template theo dõi để ghi nhận chi tiết tiến độ dự án. |
| Thứ Sáu | Lên ý tưởng sơ bộ & Kế hoạch Kiến trúc | 7.0h | Hoàn thành | Phác thảo ý tưởng về trợ lý y tế số, vạch ra các dịch vụ cloud cần thiết cho dự án. |

### Kết quả Kỹ thuật Đạt được
- **Thành thạo AWS:** Làm quen với giao diện AWS Console và các công cụ quản lý chi phí.
- **Định hướng Dự án:** Thống nhất sử dụng công nghệ Generative AI cho lĩnh vực y tế.

> [!TIP]
> **Kỷ luật Kỹ thuật:** Toàn bộ mã nguồn và cấu hình AWS đều được kiểm thử gắt gao ở local trước khi đẩy lên cloud. Mọi sự cố phát sinh trong quá trình tích hợp (ví dụ: lỗi cấp quyền IAM, xung đột package) đều được ghi chú lại để hoàn thiện các quy trình CI/CD sau này.
