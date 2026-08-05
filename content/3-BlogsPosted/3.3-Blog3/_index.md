---
title: "Blog 3: Metadata Filtering in RAG"
weight: 30
---

# Metadata Filtering: Why Filtering Before Searching is Crucial in RAG

*AWS Study Group Sharing – First Cloud AI Journey (FCAJ)*  
*Based on: [Amazon Bedrock Knowledge Bases now supports metadata filtering to improve retrieval accuracy](https://aws.amazon.com/blogs/machine-learning/amazon-bedrock-knowledge-bases-now-supports-metadata-filtering-to-improve-retrieval-accuracy/) – AWS ML Blog*

---

## 1. The Blind Spot of Similarity Search

In RAG (Retrieval-Augmented Generation), the quality of the LLM's response depends almost entirely on the quality of the retrieved context. However, vector similarity search has a blind spot: it strictly measures "semantic closeness" but struggles to inherently understand the **applicable context** of the text.

While building **Pedix**-a pediatric AI triage system for children aged 0–5-we encountered this exact problem in its most dangerous form: a guideline explaining how to manage fever for a 4-year-old can be highly "semantically close" in vector space to a query about a 6-day-old newborn with a fever. Yet, the clinical management for these two ages is vastly different (home monitoring vs. immediate emergency care). If retrieval relies purely on semantic similarity, the risk of misaligned and dangerous advice is very real.

## 2. What Metadata Filtering Solves

The AWS ML Blog introduced metadata filtering for Bedrock Knowledge Bases: instead of executing a similarity search across the entire vector store and hoping the results match the right context, you attach **metadata** (dates, categories, sources...) to each chunk. 

Then, you **pre-filter** the candidate set based on that metadata *before* running the similarity search on the remaining subset. The result: retrieval is both faster (due to a smaller search space) and much more accurate (eliminating noise early rather than forcing the LLM to sift it out).

![Blog 3 Illustration](/images/blogs/blog3.jpg)

## 3. Practical Implementation in Pedix

Pedix doesn't use managed Bedrock Knowledge Bases; instead, we self-host Qdrant on EC2. However, the underlying principle is identical. Every chunk in the knowledge base is tagged with 5 metadata fields, the most critical being `age_group`:

```json
{
  "age_group": "newborn | young_infant | infant | toddler | preschool | all",
  "urgency_relevance": "emergency | urgent | routine",
  "source_authority": "WHO | NICE | CDC | AAP",
  "content_type": "triage_protocol | symptom_cluster | care_pathway | parent_education"
}
```

Our retrieval pipeline runs in 2 passes:

- **Pass 1 – Hard Filter:** Query Qdrant with the filter `age_group in [current_age_group, "all"]` **before** calculating similarity. A newborn profile will never retrieve chunks intended exclusively for a 4-year-old, regardless of how high the similarity score might be.
- **Pass 2 – Rerank:** A cross-encoder rescores this filtered set based on symptom matching, age context understanding, and source authority confidence.

```python
chunks = retriever.retrieve(
    embedding=symptom_embedding,
    filter={"age_group": {"$in": [age_group, "all"]}}
)
chunks = reranker.rerank(chunks, symptom_summary)
```

## 4. Key Takeaways

My biggest realization from implementing this part: **metadata filtering isn't just a performance optimization; in many sensitive domains, it is a mandatory business-logic guardrail.** 

For pediatric healthcare, hard-filtering by age before searching isn't a "nice to have"-it is a prerequisite to ensure the system doesn't generate contextually incorrect advice. If you rely solely on semantic search and hope the LLM is "smart enough to understand the age context," the risk remains. If the retrieval is wrong, even the smartest LLM will reason on bad data.

**Pro-tip:** The fields used for filtering should be designed **right at the ingestion phase**, not retrofitted later. Tagging `age_group` onto ~1,800 chunks during the initial knowledge base build was a tedious upfront effort, but it paid off massively by eliminating complex runtime logic in the retrieval pipeline.

---

**Reference:**
- Corvus Lee – *Amazon Bedrock Knowledge Bases now supports metadata filtering to improve retrieval accuracy*, AWS ML Blog. [Link](https://aws.amazon.com/blogs/machine-learning/amazon-bedrock-knowledge-bases-now-supports-metadata-filtering-to-improve-retrieval-accuracy/)