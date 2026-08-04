---
title: "Week 11: Security Roles & AI Model Configuration"
date: 2026-05-11
weight: 11
chapter: false
pre: " <b> 1.11. </b> "
---

### Week 11 Summary
Finalized cloud architecture security layers. Configured granular IAM policies and evaluated Bedrock model throughput limits, leading to the selection of Claude Haiku for optimal cost/performance.

### Daily Worklog & Technical Notes
| Day | Task Description | Duration | Status | Technical Notes |
| :--- | :--- | :--- | :--- | :--- |
| Monday | Local E2E Verification Part 1 | 8.0h | Done | Performed massive integration testing. Verified UI workflows and the Agentic RAG reasoning traces locally. |
| Tuesday | Local E2E Verification Part 2 | 7.0h | Done | Verified DynamoDB mock integrations and Cognito authentication pathways before transitioning to cloud. |
| Wednesday | IAM Least Privilege Setup | 7.5h | Done | Created `Pedix-EC2-Role` limiting DynamoDB access. Eliminated all hardcoded credentials from the EC2 instance. |
| Thursday | AWS Cloud Environment Check | 9.0h | Done | Validated AWS region quotas and limits in `ap-southeast-1` to ensure sufficient resources for production deployment. |
| Friday | Bedrock Model Transition | 8.5h | Done | Switched from Claude 3.5 Sonnet to Claude 3 Haiku to mitigate severe Free Tier throttling (ThrottlingException). |

### Key Outcomes & Deliverables
- **Security Compliance:** 100% IAM Least Privilege enforcement.
- **Stability:** Mitigated AWS Free Tier AI throttling limits.

> [!TIP]
> **Engineering Discipline:** All code and AWS configurations were rigorously tested locally before deployment. Issues encountered during integration (e.g., IAM permission faults, package dependencies) were documented to refine future CI/CD pipelines.
