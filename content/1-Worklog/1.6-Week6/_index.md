---
title: "Week 6: UI Design & Service Orchestration"
date: 2026-05-11
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

### Week 6 Summary
Shifted focus to the frontend user experience. Designed a responsive, mobile-first React application using TailwindCSS and set up Docker Compose to orchestrate local backend services.

### Daily Worklog & Technical Notes
| Day | Task Description | Duration | Status | Technical Notes |
| :--- | :--- | :--- | :--- | :--- |
| Monday | React UI Scaffolding | 8.0h | Done | Initialized React SPA using Vite. Configured TailwindCSS and set up React Router for core pages. |
| Tuesday | Chatbot UI Implementation Part 1 | 6.5h | Done | Designed a collapsible sidebar for the 'Reasoning Trace' allowing users to inspect AI decision stages. |
| Wednesday | Chatbot UI Implementation Part 2 | 8.0h | Done | Implemented auto-scrolling message bubbles, typing indicators, and markdown rendering support. |
| Thursday | Local Service Orchestration | 6.5h | Done | Created `docker-compose.yml` mapping FastAPI (8000) and Qdrant (6333). Ensured persistent volume mapping. |
| Friday | Frontend-Backend Wiring | 8.0h | Done | Connected React Axios client to FastAPI endpoints. Handled CORS preflight `OPTIONS` requests correctly. |

### Key Outcomes & Deliverables
- **Functional UI:** A polished, mobile-responsive chat interface.
- **Orchestration:** 1-click local launch via Docker Compose.

> [!TIP]
> **Engineering Discipline:** All code and AWS configurations were rigorously tested locally before deployment. Issues encountered during integration (e.g., IAM permission faults, package dependencies) were documented to refine future CI/CD pipelines.
