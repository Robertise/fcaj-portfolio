---
title : "Introduction"
date : 2024-01-01 
weight : 1 
chapter : false
pre : " <b> 5.1. </b> "
---

#### Pedix Architecture & AWS Services

**Pedix** is an Age-Stratified Level 4 Agentic Retrieval-Augmented Generation (RAG) system built on AWS cloud infrastructure.

The system connects 12 core AWS services into a resilient, highly secure, and cost-optimized production stack:
+ **Frontend:** React (Vite) static build hosted on **Amazon S3** and distributed worldwide via **Amazon CloudFront** with Origin Access Control (OAC).
+ **Authentication:** User identity & group management powered by **Amazon Cognito** and **AWS Lambda** (Post-Confirmation trigger).
+ **API & Security:** Public traffic flows into **Amazon API Gateway (REST API)**, routed through **VPC Link V2** to an **Internal Application Load Balancer (ALB)**.
+ **Backend Compute:** **Amazon EC2 (t3.micro)** hosts the FastAPI backend server, sentence-transformers embedding models, and **Qdrant Vector DB** running inside Docker.
+ **Generative AI:** Clinical query analysis and multi-stage reasoning powered by **Amazon Bedrock (Claude Sonnet & Haiku)**.
+ **Persistence & Operations:** Session and analytics records stored in **Amazon DynamoDB**, with centralized logs and alarms managed by **Amazon CloudWatch** and **AWS Budgets**.

#### Architecture Diagram

![Pedix AWS Cloud Architecture](/images/5-Workshop/5.1-Workshop-overview/diagram1.png)