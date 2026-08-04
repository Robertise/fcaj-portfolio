---
title: "Week 3: Knowledge Base Curation & Architecture Drafting"
date: 2026-05-11
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---

### Week 3 Summary
Focused on data preparation and system design. Extracted and cleaned thousands of lines of clinical guidelines from WHO PDFs using MinerU. Drafted the initial cloud architecture diagram.

### Daily Worklog & Technical Notes
| Day | Task Description | Duration | Status | Technical Notes |
| :--- | :--- | :--- | :--- | :--- |
| Monday | WHO Guidelines Extraction | 7.5h | Done | Extracted text from 'Pocket book of hospital care for children' using MinerU. Handled complex OCR issues. |
| Tuesday | Document Structuring & Cleaning | 8.5h | Done | Manually formatted markdown tables, lists, and headers to ensure high-quality text representation. |
| Wednesday | Semantic Chunking Strategy | 8.0h | Done | Researched Contextual Retrieval. Designed a Markdown-aware chunker (300-500 tokens) with 50-token overlap. |
| Thursday | Draft System Architecture | 6.5h | Done | Mapped out AWS services: API Gateway, EC2 for Backend/Qdrant, Bedrock, and DynamoDB (Free Tier optimized). |
| Friday | Zero-Trust Network Design | 7.0h | Done | Planned security group chaining to ensure the EC2 backend is fully isolated from direct internet access. |

### Key Outcomes & Deliverables
- **Data Readiness:** Converted 3 complex medical PDF books into clean, chunkable Markdown files.
- **Architecture Vision:** Established a clear blueprint for a secure, low-cost AWS deployment.

> [!TIP]
> **Engineering Discipline:** All code and AWS configurations were rigorously tested locally before deployment. Issues encountered during integration (e.g., IAM permission faults, package dependencies) were documented to refine future CI/CD pipelines.
