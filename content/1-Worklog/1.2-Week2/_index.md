---
title: "Week 2: RAG Research & Pediatric Scope Definition"
date: 2026-05-11
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---

### Week 2 Summary
Deep dive into Retrieval-Augmented Generation (RAG) concepts. After consulting with mentors, the project scope was narrowed strictly to a pediatric healthcare assistant for children under 5 (Pedix).

### Daily Worklog & Technical Notes
| Day | Task Description | Duration | Status | Technical Notes |
| :--- | :--- | :--- | :--- | :--- |
| Monday | Research Agentic RAG Frameworks | 7.5h | Done | Explored Level 1-4 Agentic RAG. Designed a transparent multi-stage reasoning trace (Plan -> Retrieve -> Reflect -> Output). |
| Tuesday | Vector Database Comparisons | 8.0h | Done | Compared Pinecone, OpenSearch, and Qdrant. Selected Qdrant for local Docker support and low memory overhead. |
| Wednesday | Healthcare Data Categorization | 6.5h | Done | Organized general medical data into 14 major categories before realizing the scope was too broad. |
| Thursday | Consultation with Mentor | 7.5h | Done | Discussed project scope with tutor. Pivoted to pediatric care (under-5) to leverage strict clinical thresholds. |
| Friday | Pediatric Scope Definition | 7.0h | Done | Outlined specific age thresholds (newborn, infant, toddler) and mapped initial clinical urgency levels. |

### Key Outcomes & Deliverables
- **Architectural Decision:** Adopted a Level 4 Agentic RAG approach.
- **Data Strategy:** Finalized the decision to exclusively use WHO and NICE pediatric guidelines.

> [!TIP]
> **Engineering Discipline:** All code and AWS configurations were rigorously tested locally before deployment. Issues encountered during integration (e.g., IAM permission faults, package dependencies) were documented to refine future CI/CD pipelines.
