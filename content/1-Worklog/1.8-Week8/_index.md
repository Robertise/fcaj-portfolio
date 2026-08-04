---
title: "Week 8: Batch Ingestion & DynamoDB Refactoring"
date: 2026-05-11
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---

### Week 8 Summary
Finalized the data ingestion pipeline, successfully embedding all clinical guidelines. Completely refactored the backend database layer to interface cleanly with Amazon DynamoDB On-Demand tables.

### Daily Worklog & Technical Notes
| Day | Task Description | Duration | Status | Technical Notes |
| :--- | :--- | :--- | :--- | :--- |
| Monday | Batch Ingestion Script | 8.0h | Done | Wrote a Python script utilizing `sentence-transformers` (CPU build) to batch embed WHO guidelines. |
| Tuesday | Qdrant Embeddings Processing | 8.0h | Done | Executed the batch ingestion, mapping 14 clinical categories into the `pedix_kb` collection. |
| Wednesday | DynamoDB Refactoring Part 1 | 9.0h | Done | Replaced mock storage with Boto3 DB operations. Implemented efficient `Query` and `PutItem` functions. |
| Thursday | Asynchronous DB Handling | 6.5h | Done | Wrapped blocking Boto3 calls in FastAPI's `run_in_threadpool` to prevent ASGI event loop blocking. |
| Friday | DynamoDB Keys Design | 6.5h | Done | Designed Composite Primary Keys (Partition Key: `user_id`, Sort Key: `profile_id`) to avoid costly `Scan` queries. |

### Key Outcomes & Deliverables
- **Database Ready:** Backend fully integrated with production-ready DynamoDB patterns.
- **Knowledge Embedded:** 10,000+ lines of guidelines vectorized and searchable.

> [!TIP]
> **Engineering Discipline:** All code and AWS configurations were rigorously tested locally before deployment. Issues encountered during integration (e.g., IAM permission faults, package dependencies) were documented to refine future CI/CD pipelines.
