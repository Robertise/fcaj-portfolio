---
title: "Week 10: Rebranding & Analytics Module"
date: 2026-05-11
weight: 10
chapter: false
pre: " <b> 1.10. </b> "
---

### Week 10 Summary
Officially rebranded the project to 'Pedix'. Built an admin analytics dashboard utilizing parallel DynamoDB querying and implemented strict Role-Based Access Control (RBAC) via Cognito.

### Daily Worklog & Technical Notes
| Day | Task Description | Duration | Status | Technical Notes |
| :--- | :--- | :--- | :--- | :--- |
| Monday | Codebase Rebranding | 9.0h | Done | Renamed packages, environment variables, and docs from PediCompass to Pedix to reflect final branding. |
| Tuesday | RBAC via Cognito Groups | 8.5h | Done | Created `pedix-users` & `pedix-admins` groups. Added FastAPI Dependency `get_admin_user` to reject bad JWTs. |
| Wednesday | Analytics Dashboard UI | 7.5h | Done | Created the frontend views for the admin dashboard, including metric cards and session log tables. |
| Thursday | Parallel Analytics Queries | 7.0h | Done | Used Python's `asyncio.gather` to query DynamoDB tables simultaneously, reducing dashboard load time by 60%. |
| Friday | Pre-Visit Checklist Feature | 7.5h | Done | Implemented custom UI markdown parsing to explicitly extract and highlight the 'Things to bring' output from Stage 5. |

### Key Outcomes & Deliverables
- **Admin Controls:** Secure, RBAC-protected administrative views.
- **Performance:** Highly optimized DynamoDB data aggregation for dashboard metrics.

> [!TIP]
> **Engineering Discipline:** All code and AWS configurations were rigorously tested locally before deployment. Issues encountered during integration (e.g., IAM permission faults, package dependencies) were documented to refine future CI/CD pipelines.
