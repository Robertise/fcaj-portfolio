---
title: "Week 12: Zero-Trust AWS Deployment & Verification"
date: 2026-05-11
weight: 12
chapter: false
pre: " <b> 1.12. </b> "
---

### Week 12 Summary
Executed the live production deployment. Orchestrated a Zero-Trust network combining API Gateway VPC Link, Internal ALB, and Cognito. The system went fully live on a CloudFront domain.

### Daily Worklog & Technical Notes
| Day | Task Description | Duration | Status | Technical Notes |
| :--- | :--- | :--- | :--- | :--- |
| Monday | EC2, ALB & Qdrant Deploy | 6.5h | Done | Provisioned EC2 `t3.micro` with 2GB Swap. Configured Internal ALB. Started Qdrant and FastAPI backend services. |
| Tuesday | API Gateway & VPC Link Setup | 7.0h | Done | Configured REST API Gateway with Cognito. Bridged traffic privately to the ALB via VPC Link V2 (`fzvy02`). |
| Wednesday | S3 & CloudFront CDN Deploy | 6.5h | Done | Built React SPA (`npm run build`). Created CloudFront distribution with Origin Access Control (OAC) and SPA fallbacks. |
| Thursday | End-to-End Live Testing | 8.0h | Done | Conducted live testing on the public CloudFront domain. Verified API health checks and SSE streaming stability. |
| Friday | Project Verification & Slides | 7.5h | Done | Compiled all technical artifacts, verified project alignment with requirements, and finalized presentation slides. |

### Key Outcomes & Deliverables
- **Live Production:** System successfully deployed and publicly accessible via CloudFront.
- **Zero-Trust Achieved:** EC2 backend is fully isolated from public internet access, accepting traffic exclusively through the API Gateway VPC Link.

> [!TIP]
> **Engineering Discipline:** All code and AWS configurations were rigorously tested locally before deployment. Issues encountered during integration (e.g., IAM permission faults, package dependencies) were documented to refine future CI/CD pipelines.
