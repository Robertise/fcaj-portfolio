---
title: "Blog 2"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.2. </b> "
---
{{% notice warning %}}
⚠️ **Note:** The information below is for reference purposes only. Please **do not copy verbatim** for your report, including this warning.
{{% /notice %}}

# COST-OPTIMIZED VECTOR DB PRE-FILTERING WITH QDRANT ON AWS EC2

### Introduction
Building vector search infrastructure for RAG applications often leads to high cloud bills when managed serverless vector databases (such as OpenSearch Serverless) are selected for small-to-medium workloads. This blog demonstrates how hosting **Qdrant** in a Docker container on an **Amazon EC2 (t3.micro)** instance paired with payload indexing achieved sub-50ms query latencies at a fraction of the cost (~$9.50/month EC2 compute).

---

### Key Optimization Strategies

#### 1. Payload Indexing for Mandatory Pre-Filtering
Standard vector search algorithms calculate cosine similarity across all vectors before filtering metadata (post-filtering), causing high CPU usage and irrelevant neighbor retrieval. By creating a Qdrant KEYWORD payload index on `age_group`:
```python
client.create_payload_index(
    collection_name="pedix_kb",
    field_name="age_group",
    field_schema=PayloadSchemaType.KEYWORD
)
```
Qdrant executes payload filters *before* vector distance calculations, reducing search candidate pools by up to 80% and decreasing query latency to under **35ms**.

#### 2. Embedding Model Sizing & PyTorch CPU Optimization
Instead of heavy GPU instances, Pedix uses `all-MiniLM-L6-v2` producing 384-dimensional vectors. The backend installs PyTorch CPU-only builds (`whl/cpu`), reducing memory footprint and allowing embedding generation directly on EC2.

#### 3. EBS Swap Configuration for Memory Stability
To prevent Out-Of-Memory (OOM) crashes on 2GB RAM t3.micro instances, a 2GB EBS Swap file was configured. This guarantees headroom for cross-encoder reranking models during burst requests.