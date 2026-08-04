---
title: "Blog 1"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# LEVEL 4 AGENTIC RAG VỚI AMAZON BEDROCK & CLAUDE SONNET

### Tóm tắt điều hành
Retrieval-Augmented Generation (RAG) đã trở thành công nghệ cốt lõi cho các ứng dụng AI doanh nghiệp. Tuy nhiên, trong lĩnh vực đòi hỏi độ chính xác tuyệt đối như y tế nhi khoa, các RAG pipeline thông thường dễ gặp lỗi ảo giác (hallucination) và thiếu tính minh bạch lâm sàng. Bài viết này phân tích cách xây dựng kiến trúc **Agentic RAG Level 4 Phân tầng Độ tuổi** ứng dụng **Amazon Bedrock (Claude Sonnet & Haiku)** cho hệ thống **Pedix**.

---

### Các Trụ cột Kiến trúc Cốt lõi

#### 1. Màng Lọc An Toàn Định Tính (Stage 0)
Trước khi gọi các mô hình LLM đắt đỏ, chuỗi truy vấn được đánh giá qua bộ phân tích regex siêu tốc (<10ms) kết hợp kiểm tra ngữ cảnh qua Claude Haiku. Nếu phát hiện các dấu hiệu cấp cứu đe dọa tính mạng ở trẻ (như tím tái, lơ mơ, sốt ở trẻ dưới 90 ngày tuổi), hệ thống lập tức chuyển hướng sang hướng dẫn cấp cứu mà không cần lặp LLM.

#### 2. Lọc Metadata Metadata Pre-filtering Phân tầng Độ tuổi (Stage 1 & 2)
Độ tuổi là biến số lâm sàng quan trọng nhất trong chăm sóc nhi khoa. Pedix tự động tính tuổi chính xác của trẻ theo ngày và ánh xạ vào các nhóm tuổi (`newborn`, `young_infant`, `infant`, `toddler`, `preschool`). Truy vấn vector trên Qdrant bắt buộc thi hành bộ lọc payload theo `age_group`, ngăn chặn các đoạn tri thức của người lớn làm nhiễu ngữ cảnh.

#### 3. Suy luận Lâm sàng Cấu trúc qua Bedrock Tool-Use (Stage 3)
Tận dụng tính năng `tool_use` của Claude Sonnet trên Amazon Bedrock, Agent đánh giá bằng chứng y khoa dựa trên khung phân loại khẩn cấp **Emergency Severity Index (ESI v4)**. Kết quả trả về là JSON cấu trúc bao gồm:
- Mức độ khẩn cấp (Emergency, Urgent, Soon, Routine)
- Lý giải lâm sàng
- Các bước lộ trình chăm sóc (Care Pathway)
- Danh mục kiểm tra trước khi đi khám

#### 4. Vòng lặp Reflection & Kiểm thử Độc lập (Stage 4 & 5)
Stage 4 kiểm tra tính đầy đủ và nguồn trích dẫn của kết quả. Nếu thiếu thông tin quan trọng, orchestrator sẽ kích hoạt tối đa 2 vòng lặp tìm kiếm bổ sung trước khi sinh câu trả lời hoàn chỉnh, an tâm cho phụ huynh.

---

### Giá trị Kỹ thuật & Kinh doanh
- **Minh bạch Hoàn toàn:** Hiển thị vết suy luận (Reasoning Trace) từng bước giúp phụ huynh tin tưởng tuyệt đối.
- **Tối ưu Chi phí:** Phối hợp linh hoạt giữa Haiku cho tác vụ nhẹ và Sonnet cho suy luận phức tạp giúp chi phí mỗi truy vấn chỉ dưới **$0.015 / request**.