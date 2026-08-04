---
title: "Blogs Posted"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 3. </b> "
---

This section lists technical blogs published on the [AWS Study Group](https://www.facebook.com/groups/awsstudygroupfcj) during my internship journey:

### [Blog 1 - LEVEL 4 AGENTIC RAG WITH AMAZON BEDROCK & CLAUDE SONNET](3.1-Blog1/)
An in-depth exploration of Level 4 Agentic Retrieval-Augmented Generation (RAG) for pediatric healthcare. Discusses how age-stratified prompt engineering, Amazon Bedrock Tool-Use APIs, and reflection loops eliminate AI black-box decisions in high-stakes clinical domains.

### [Blog 2 - COST-OPTIMIZED VECTOR DB PRE-FILTERING WITH QDRANT ON AWS EC2](3.2-Blog2/)
Demonstrates how running Qdrant as a Docker container on EC2 (t3.micro) with payload indexing on `age_group` metadata reduces AWS infrastructure costs by ~$60/month compared to OpenSearch Serverless while achieving sub-50ms vector query latencies.

### [Blog 3 - SECURING PRIVATE BACKENDS WITH API GATEWAY VPC LINK V2 & INTERNAL ALB](3.3-Blog3/)
A step-by-step guide to building zero-trust serverless backend connections on AWS using API Gateway REST proxy integrations, VPC Link V2, internal ALBs, and configuring Server-Sent Events (SSE) streaming with heartbeat keep-alives.