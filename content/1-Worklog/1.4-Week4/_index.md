---
title: "Week 4: Proposal Development & System Modeling"
date: 2026-05-11
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

### Week 4 Summary
Formalized the project proposal for the bootcamp. Finalized the architectural diagrams and explicitly defined the 5-stage reasoning loop for the Pedix assistant.

### Daily Worklog & Technical Notes
| Day | Task Description | Duration | Status | Technical Notes |
| :--- | :--- | :--- | :--- | :--- |
| Monday | Proposal Documentation Part 1 | 7.5h | Done | Wrote the executive summary and problem statement. Highlighted the dangers of generic symptom checkers. |
| Tuesday | Proposal Documentation Part 2 | 7.0h | Done | Defined expected outcomes, milestones, and budget estimations ensuring adherence to AWS Free Tier. |
| Wednesday | Stage 0 Safety Screen Design | 7.5h | Done | Designed a deterministic, sub-10ms regex/keyword filter to immediately flag critical symptoms (e.g., cyanosis) before LLM. |
| Thursday | Architecture Diagram Visualization | 7.5h | Done | Visualized the request flow from CloudFront -> API Gateway -> VPC Link -> ALB -> EC2 via Draw.io. |
| Friday | Peer Review & Logic Refinement | 8.0h | Done | Discussed the proposal with the team. Refined the ESI v4 triage mapping logic to better fit pediatric constraints. |

### Key Outcomes & Deliverables
- **Official Proposal:** Completed and approved project proposal.
- **Technical Blueprint:** Clear architectural diagrams ready for execution.

> [!TIP]
> **Engineering Discipline:** All code and AWS configurations were rigorously tested locally before deployment. Issues encountered during integration (e.g., IAM permission faults, package dependencies) were documented to refine future CI/CD pipelines.
