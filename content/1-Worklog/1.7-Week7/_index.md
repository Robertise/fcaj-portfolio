---
title: "Week 7: Pipeline Optimization & Authentication Flow"
date: 2026-05-11
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---

### Week 7 Summary
Enhanced the accuracy of the Qdrant retrieval pipeline by implementing metadata payload filtering. Began scaffolding the Amazon Cognito integration for secure user authentication.

### Daily Worklog & Technical Notes
| Day | Task Description | Duration | Status | Technical Notes |
| :--- | :--- | :--- | :--- | :--- |
| Monday | Age-Stratified Qdrant Filtering | 8.5h | Done | Implemented KEYWORD payload indexing. Stage 2 strictly filters vectors based on Stage 1 age extraction. |
| Tuesday | Stage 1 & 2 Logic Refinement | 7.0h | Done | Fine-tuned the LLM to output consistent payload keys (e.g., 'newborn', 'toddler') for Qdrant matching. |
| Wednesday | Document Parser Upgrades | 9.0h | Done | Upgraded ingestion script using PyMuPDF to preserve markdown tables during extraction, improving vectors. |
| Thursday | Cognito Authentication Mock | 7.5h | Done | Researched AWS Cognito flows. Implemented a mock JWT validator in FastAPI middleware to prepare integration. |
| Friday | Profile Context Injection | 7.5h | Done | Modified chat endpoint to auto-inject the child's profile (weight, preexisting conditions) into the Bedrock prompt. |

### Key Outcomes & Deliverables
- **Retrieval Precision:** Eliminated hallucinations caused by retrieving older-child clinical advice for infants.
- **Context Awareness:** Chatbot natively understands the patient's medical background.

> [!TIP]
> **Engineering Discipline:** All code and AWS configurations were rigorously tested locally before deployment. Issues encountered during integration (e.g., IAM permission faults, package dependencies) were documented to refine future CI/CD pipelines.
