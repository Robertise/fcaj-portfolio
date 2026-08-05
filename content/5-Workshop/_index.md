---
title: "Workshop"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5. </b> "
---
# Deploying an Agentic RAG Pediatric Health Navigator on AWS

#### Overview

In this workshop, you will learn how to build, configure, and deploy **Pedix** - a production-grade Level 4 Agentic Retrieval-Augmented Generation (RAG) system for pediatric healthcare navigation on AWS.

You will learn how to architect and deploy a secure, cost-optimized production infrastructure combining 12 AWS services:
+ **Compute & Vector DB:** Hosting FastAPI and Qdrant Vector DB on Amazon EC2 (t3.micro) with persistent EBS storage and 2GB Swap memory.
+ **AI Inference:** Invoking Amazon Bedrock Claude Sonnet for clinical reasoning and Haiku for contextual ingestion.
+ **API & Security Layer:** Exposing private EC2 endpoints using API Gateway Regional REST APIs, VPC Link V2, and Internal Application Load Balancers (ALB).
+ **Authentication & Data:** Managing users with Amazon Cognito, Lambda Post-Confirmation triggers, and Amazon DynamoDB On-Demand tables.
+ **Frontend Delivery:** Deploying React static assets to Amazon S3 with CloudFront CDN delivery and Origin Access Control (OAC).

#### Team Contributions

This project was built collaboratively by a two-person team. The workload and contributions were divided equally (50-50).

| Team Member | Status | Core Roles & Responsibilities |
| :--- | :--- | :--- |
| **Đỗ Gia Huy** | FCAJ Intern | **Pedix Lead Developer & Cloud Implementation Engineer**<br>- Solely responsible for developing the Pedix backend, AI logic, and frontend.<br>- Executed the cloud deployment, integrated the system architecture, and authored the technical reports. |
| **Huỳnh Đoàn Hoàng Minh** | Bootcamp Participant | **Pedix Supporter & Cloud Architect**<br>- Designed the overarching cloud architecture and conceptualized core system ideas.<br>- Provided support and testing for the Pedix development lifecycle. |

#### Content

1. [Workshop Overview](5.1-Workshop-overview/)
2. [Prerequisites](5.2-Prerequisites/)
3. [EC2, Qdrant & FastAPI Backend Setup](5.3-EC2-Backend/)
4. [API Gateway, VPC Link V2 & Internal ALB Configuration](5.4-API-Gateway-ALB/)
5. [Cognito Auth & DynamoDB Schema Deployment](5.5-Cognito-DynamoDB/)
6. [Frontend S3/CloudFront Deployment & System Validation](5.6-Frontend-CloudFront/)