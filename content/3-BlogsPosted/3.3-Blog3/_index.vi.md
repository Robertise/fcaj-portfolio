---
title: "Blog 3: Metadata Filtering trong RAG"
weight: 30
---

# Metadata Filtering: Vì Sao Lọc Trước Khi Tìm Kiếm Lại Quan Trọng Hơn Cố Tìm Kiếm Giỏi

*Bài chia sẻ AWS Study Group – First Cloud AI Journey (FCAJ)*  
*Dựa trên: [Amazon Bedrock Knowledge Bases now supports metadata filtering to improve retrieval accuracy](https://aws.amazon.com/blogs/machine-learning/amazon-bedrock-knowledge-bases-now-supports-metadata-filtering-to-improve-retrieval-accuracy/) – AWS ML Blog*

---

## 1. Vấn đề của Tìm kiếm Tương đồng (Similarity Search)

Trong RAG (Retrieval-Augmented Generation), chất lượng câu trả lời của LLM phụ thuộc gần như hoàn toàn vào chất lượng context được retrieve ra. Nhưng vector similarity search có một điểm mù: nó chỉ đo "gần nghĩa" chứ không hiểu **ngữ cảnh áp dụng** của đoạn văn bản đó.

Khi làm **Pedix** – hệ thống AI triage nhi khoa cho trẻ 0–5 tuổi – nhóm mình gặp đúng vấn đề này ở dạng nguy hiểm nhất: một đoạn hướng dẫn xử lý sốt cho trẻ 4 tuổi có thể "gần nghĩa" về mặt embedding với câu hỏi về trẻ sơ sinh 6 ngày tuổi bị sốt. Dù vậy, nguyên tắc xử lý lâm sàng giữa hai độ tuổi này khác nhau hoàn toàn (một bên là theo dõi tại nhà, một bên là cấp cứu ngay). Nếu retrieval chỉ dựa vào similarity thuần, nguy cơ trả lời sai lệch là có thật.

## 2. Metadata Filtering giải quyết gì

Bài viết trên AWS ML Blog giới thiệu tính năng metadata filtering cho Bedrock Knowledge Bases: thay vì để similarity search chạy trên toàn bộ vector store rồi hy vọng kết quả đúng ngữ cảnh, bạn gắn **metadata** (ngày tháng, danh mục, nguồn...) cho từng chunk. 

Sau đó, bạn **lọc trước (pre-filter)** tập ứng viên theo metadata đó *trước khi* chạy similarity search trên tập đã lọc. Kết quả: retrieval vừa nhanh hơn (không gian search nhỏ hơn) vừa chính xác hơn (loại bỏ nhiễu ngay từ đầu thay vì để LLM tự "đãi cát tìm vàng").

![Blog 3 Illustration](/images/blogs/blog3.jpg)

## 3. Thực nghiệm áp dụng vào Pedix

Pedix không dùng Bedrock Knowledge Bases managed mà tự host Qdrant trên EC2. Tuy nhiên, nguyên lý áp dụng y hệt. Mỗi chunk trong knowledge base được gắn 5 trường metadata, quan trọng nhất là `age_group`:

```json
{
  "age_group": "newborn | young_infant | infant | toddler | preschool | all",
  "urgency_relevance": "emergency | urgent | routine",
  "source_authority": "WHO | NICE | CDC | AAP",
  "content_type": "triage_protocol | symptom_cluster | care_pathway | parent_education"
}
```

Retrieval pipeline của nhóm chạy 2 pass:

- **Pass 1 – Hard filter:** query Qdrant với filter `age_group in [nhóm_tuổi_hiện_tại, "all"]` **trước** khi tính similarity. Trẻ sơ sinh sẽ không bao giờ nhìn thấy chunk chỉ dành cho trẻ 4 tuổi, bất kể điểm similarity cao đến đâu.
- **Pass 2 – Rerank:** cross-encoder chấm điểm lại tập đã lọc theo mức trúng khớp triệu chứng + độ đặc hiểu tuổi + độ tin cậy nguồn.

```python
chunks = retriever.retrieve(
    embedding=symptom_embedding,
    filter={"age_group": {"$in": [age_group, "all"]}}
)
chunks = reranker.rerank(chunks, symptom_summary)
```

## 4. Bài học rút ra

Điểm mình tâm đắc nhất sau khi làm phần này: **metadata filtering không phải là tính năng tối ưu hiệu năng, mà nhiều khi là một guardrail bắt buộc về mặt nghiệp vụ.** 

Với domain nhạy cảm như y tế nhi khoa, việc filter cứng theo tuổi trước khi search không phải "nice to have" – nó là điều kiện tiên quyết để hệ thống không đưa ra khuyến nghị sai ngữ cảnh. Nếu chỉ dựa vào semantic search + hy vọng LLM "đủ thông minh để tự hiểu ngữ cảnh tuổi", rủi ro vẫn còn đó. Nếu retrieval sai thì LLM có giỏi đến đâu cũng chỉ reasoning trên dữ liệu sai.

**Mẹo nhỏ học được:** field dùng để filter cứng nên được thiết kế **ngay từ lúc ingest**, không phải chắp vá vào sau. Việc gắn `age_group` cho ~1.800 chunks lúc build knowledge base tốn nhiều công sức ban đầu, nhưng đổi lại retrieval pipeline không cần logic phức tạp gì thêm ở runtime.

---

**Tham khảo:**
- Corvus Lee – *Amazon Bedrock Knowledge Bases now supports metadata filtering to improve retrieval accuracy*, AWS ML Blog. [Link](https://aws.amazon.com/blogs/machine-learning/amazon-bedrock-knowledge-bases-now-supports-metadata-filtering-to-improve-retrieval-accuracy/)