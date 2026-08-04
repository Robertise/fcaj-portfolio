---
title: "Các bài blogs đã đăng"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 3. </b> "
---

Mục này tổng hợp các bài blog kỹ thuật đã đăng trên cộng đồng [AWS Study Group](https://www.facebook.com/groups/awsstudygroupfcj) trong kỳ thực tập:

### [Blog 1 - LEVEL 4 AGENTIC RAG VỚI AMAZON BEDROCK & CLAUDE SONNET](3.1-Blog1/)
Phân tích chuyên sâu về kiến trúc Agentic RAG Level 4 ứng dụng trong y tế nhi khoa. Bài viết giải thích chi tiết cách phân tầng độ tuổi, lập trình prompt tool-use trên Amazon Bedrock và vòng lặp tự đánh giá (reflection) giúp loại bỏ hiện tượng "hộp đen AI".

### [Blog 2 - TỐI ƯU CHI PHÍ VECTOR DATABASE VỚI QDRANT & PRE-FILTERING TRÊN AWS EC2](3.2-Blog2/)
Hướng dẫn triển khai Qdrant Docker trên EC2 (t3.micro) kết hợp chỉ mục payload `age_group` giúp giảm ~$60/tháng chi phí hạ tầng so với OpenSearch Serverless nhưng vẫn đạt tốc độ truy vấn vector dưới 50ms.

### [Blog 3 - BẢO MẬT BACKEND NỘI BỘ VỚI API GATEWAY VPC LINK V2 & INTERNAL ALB](3.3-Blog3/)
Hướng dẫn từng bước xây dựng kết nối Zero-Trust cho backend serverless trên AWS bằng API Gateway REST proxy integration, VPC Link V2, Internal ALB và cấu hình Server-Sent Events (SSE) streaming hỗ trợ cơ chế heartbeat keep-alive.