---
title: "Week 9: SSE Streaming & ESI v4 Triage Reasoning"
date: 2026-05-11
weight: 9
chapter: false
pre: " <b> 1.9. </b> "
---

### Week 9 Summary
A critical week focused on real-time UX and complex clinical logic. Implemented Server-Sent Events (SSE) to stream the 5-stage reasoning trace to the UI, and codified ESI v4 urgency guidelines.

### Daily Worklog & Technical Notes
| Day | Task Description | Duration | Status | Technical Notes |
| :--- | :--- | :--- | :--- | :--- |
| Monday | SSE Streaming Implementation | 8.5h | Done | Converted the `/api/chat` endpoint to a `StreamingResponse`, yielding JSON progressively. |
| Tuesday | Real-time Trace Generation | 7.0h | Done | Handled UI state to incrementally reveal the AI's internal reasoning trace as stages complete. |
| Wednesday | SSE Heartbeat Keep-Alive | 7.5h | Done | Added a 5.0s empty heartbeat yield frame. Prevents CloudFront from dropping idle connections during LLM calls. |
| Thursday | ESI v4 Logic Integration | 8.0h | Done | Injected Emergency Severity Index (ESI) v4 rules into Bedrock prompt to enforce proper urgency levels (1-5). |
| Friday | Chat Intent Validation | 7.5h | Done | Added a Bedrock pre-flight check to detect and politely refuse non-medical queries, preserving API token costs. |

### Key Outcomes & Deliverables
- **Real-time UX:** Parents see the AI's 'thinking process' instantly without waiting 15 seconds.
- **Clinical Safety:** Strict adherence to ESI v4 categorization.

> [!TIP]
> **Engineering Discipline:** All code and AWS configurations were rigorously tested locally before deployment. Issues encountered during integration (e.g., IAM permission faults, package dependencies) were documented to refine future CI/CD pipelines.
