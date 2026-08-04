---
title: "Week 5: Backend Foundation & Qdrant Integration"
date: 2026-05-11
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

### Week 5 Summary
Began core software development. Bootstrapped the FastAPI backend, set up local Docker containers for Qdrant, and implemented the foundational LLM prompts using Anthropic Claude Haiku.

### Daily Worklog & Technical Notes
| Day | Task Description | Duration | Status | Technical Notes |
| :--- | :--- | :--- | :--- | :--- |
| Monday | FastAPI Project Scaffolding | 7.0h | Done | Created modular folders (api, core, services). Configured CORS middleware and global HTTP error handlers. |
| Tuesday | Database Mock Models | 7.5h | Done | Defined Pydantic schemas for User Profiles, Sessions, and Analytics. Prepared interfaces for DynamoDB. |
| Wednesday | Qdrant Vector DB Setup via Docker | 7.5h | Done | Pulled Qdrant v1.10.1 Docker image. Initialized the `pedix_kb` collection with 384-dimensional cosine vectors. |
| Thursday | Local Environment & Boto3 Config | 8.5h | Done | Configured AWS Profiles locally and set up environment variables for Bedrock API access. |
| Friday | Bedrock Prompt Engineering | 6.5h | Done | Wrote explicit XML-tagged system prompts for Query Analysis (Stage 1) and Clinical Reasoning (Stage 3). |

### Key Outcomes & Deliverables
- **Running Backend:** FastAPI server active locally at port 8000.
- **AI Connectivity:** Successfully invoked Amazon Bedrock Claude Haiku via Boto3.

> [!TIP]
> **Engineering Discipline:** All code and AWS configurations were rigorously tested locally before deployment. Issues encountered during integration (e.g., IAM permission faults, package dependencies) were documented to refine future CI/CD pipelines.
