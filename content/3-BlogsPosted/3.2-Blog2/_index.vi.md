---
title: "Blog 2"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.2. </b> "
---
{{% notice warning %}}
⚠️ **Lưu ý:** Các thông tin dưới đây chỉ nhằm mục đích tham khảo, vui lòng **không sao chép nguyên văn** cho bài báo cáo của bạn kể cả warning này.
{{% /notice %}}

# TỐI ƯU CHI PHÍ VECTOR DATABASE VỚI QDRANT & PRE-FILTERING TRÊN AWS EC2

### Giới thiệu
Việc triển khai hạ tầng tìm kiếm vector cho ứng dụng RAG thường dẫn đến chi phí đám mây tăng cao nếu chọn các dịch vụ Managed Vector DB (như OpenSearch Serverless ~$70+/tháng) cho khối lượng công việc nhỏ và vừa. Bài viết này hướng dẫn cách vận hành **Qdrant Vector DB** trong Docker trên **Amazon EC2 (t3.micro)** kết hợp kỹ thuật đánh chỉ mục payload để đạt tốc độ truy vấn dưới 50ms với chi phí cực thấp (~$9.50/tháng).

---

### Các Chiến lược Tối ưu Cốt lõi

#### 1. Đánh Chỉ Mục Payload để Pre-Filter Bắt Buộc
Thuật toán tìm kiếm vector thông thường tính khoảng cách cosine trên toàn bộ vector trước khi lọc metadata (post-filtering), gây tốn CPU và dễ kéo theo các vector không phù hợp. Bằng cách khởi tạo chỉ mục KEYWORD payload cho thuộc tính `age_group` trên Qdrant:
```python
client.create_payload_index(
    collection_name="pedix_kb",
    field_name="age_group",
    field_schema=PayloadSchemaType.KEYWORD
)
```
Qdrant thực thi lọc metadata *trước* khi tính khoảng cách vector, thu hẹp 80% không gian tìm kiếm và giảm thời gian phản hồi xuống dưới **35ms**.

#### 2. Lựa Chọn Kích Thước Vector & PyTorch CPU-Only
Thay vì sử dụng các mô hình nén nặng đòi hỏi GPU, Pedix áp dụng `all-MiniLM-L6-v2` sinh vector 384 chiều. Backend cài đặt gói PyTorch CPU-only (`whl/cpu`), giúp giảm thiểu bộ nhớ RAM và chạy trực tiếp trên EC2 t3.micro.

#### 3. Cấu Hình EBS Swap Đảm Bảo Tĩnh Sẵn Sàng
Để tránh lỗi tràn bộ nhớ (OOM crash) trên máy chủ 2GB RAM, hệ thống được cấu hình file EBS Swap 2GB. Điều này đảm bảo bù đắp dung lượng cho mô hình Cross-Encoder Reranker khi xảy ra lưu lượng truy cập cao.