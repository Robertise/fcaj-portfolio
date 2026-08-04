---
title : "Giới thiệu"
date : 2024-01-01 
weight : 1
chapter : false
pre : " <b> 5.1. </b> "
---

#### Kiến trúc Pedix & Các Dịch vụ AWS

**Pedix** là hệ thống Agentic Retrieval-Augmented Generation (RAG) Level 4 phân tầng độ tuổi y khoa được xây dựng hoàn toàn trên hạ tầng đám mây AWS.

Hệ thống kết nối 12 dịch vụ đám mây cốt lõi của AWS tạo thành một hạ tầng chuẩn Production an toàn, bảo mật và tối ưu chi phí:
+ **Giao diện Frontend:** Mã nguồn tĩnh React (Vite) lưu trữ trên **Amazon S3** và phân phối toàn cầu qua **Amazon CloudFront** với Origin Access Control (OAC).
+ **Quản lý Định danh:** Xác thực người dùng và phân quyền nhóm tự động qua **Amazon Cognito** và **AWS Lambda** (Post-Confirmation trigger).
+ **Lớp API & Bảo mật:** Luồng dữ liệu công cộng đi qua **Amazon API Gateway (REST API)**, định tuyến qua **VPC Link V2** tới **Internal Application Load Balancer (ALB)**.
+ **Máy chủ Backend:** **Amazon EC2 (t3.micro)** chạy FastAPI backend server, mô hình sentence-transformers và **Qdrant Vector DB** trong môi trường Docker.
+ **Generative AI:** Phân tích truy vấn lâm sàng và quy trình suy luận 5 giai đoạn được vận hành bởi **Amazon Bedrock (Claude Sonnet & Haiku)**.
+ **Lưu trữ & Vận hành:** Dữ liệu phiên hội thoại và phân tích lưu tại **Amazon DynamoDB**, kết hợp giám sát log và báo động chi phí từ **Amazon CloudWatch** và **AWS Budgets**.

#### Sơ đồ Kiến trúc Cloud

![Kiến trúc AWS Pedix](/images/5-Workshop/5.1-Workshop-overview/diagram1.png)
