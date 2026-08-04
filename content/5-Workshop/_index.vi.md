---
title: "Workshop"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5. </b> "
---


# Triển khai Hệ thống Agentic RAG Hướng dẫn Sức khỏe Nhi khoa trên AWS

#### Tổng quan

Trong bài workshop này, chúng ta sẽ học cách xây dựng, cấu hình và triển khai **Pedix** - một hệ thống Agentic Retrieval-Augmented Generation (RAG) Level 4 phân tầng độ tuổi y khoa chuẩn Production trên đám mây AWS.

Bạn sẽ thực hành thiết kế và triển khai hạ tầng an toàn, tối ưu chi phí kết hợp 12 dịch vụ AWS:
+ **Compute & Vector DB:** Triển khai FastAPI backend và Qdrant Vector DB trên Amazon EC2 (t3.micro) với bộ nhớ EBS lưu trữ bền vững và 2GB Swap.
+ **AI Inference:** Gọi suy luận lâm sàng qua Amazon Bedrock (Claude Sonnet) và xử lý tài liệu (Claude Haiku).
+ **Lớp API & Bảo mật:** Bảo mật endpoint EC2 nội bộ thông qua API Gateway Regional REST API, VPC Link V2 và Internal Application Load Balancer (ALB).
+ **Xác thực & Dữ liệu:** Quản lý người dùng qua Amazon Cognito, Lambda Post-Confirmation trigger và 4 bảng Amazon DynamoDB On-Demand.
+ **Phân phối Frontend:** Lưu trữ mã nguồn tĩnh React trên Amazon S3 và phân phối toàn cầu qua CloudFront CDN với Origin Access Control (OAC).

#### Nội dung

1. [Tổng quan về workshop](5.1-Workshop-overview/)
2. [Chuẩn bị](5.2-Prerequiste/)
3. [Thiết lập EC2, Qdrant & FastAPI Backend](5.3-S3-vpc/)
4. [Cấu hình API Gateway, VPC Link V2 & Internal ALB](5.4-S3-onprem/)
5. [Triển khai Cognito Auth & Bảng DynamoDB](5.5-Policy/)
6. [Phân phối Frontend S3/CloudFront & Kiểm thử hệ thống](5.6-Cleanup/)