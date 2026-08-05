---
title: "1. Workshop Overview"
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

# Deploying an Agentic RAG Pediatric Health Navigator on AWS

This workshop guides you step-by-step through deploying **Pedix**, a production-grade Agentic Retrieval-Augmented Generation (RAG) system for pediatric healthcare navigation.

By the end of this workshop, you will have built a highly secure, Zero-Trust cloud architecture leveraging 12 AWS services, fully optimized for a production environment.

---

### Architecture Diagram

![PediCompass AWS Cloud Architecture](/images/2-Proposal/pedix_architecture.png)

### Request Flow (Zero-Trust VPC Isolation)

Our architecture enforces a strict **Zero-Trust Network Isolation** design:
1. **User (Browser):** Connects to the React frontend hosted securely on **Amazon S3** via **Amazon CloudFront** CDN.
2. **API Requests:** API calls route to **Amazon API Gateway**, which automatically validates the user's JWT token using an **Amazon Cognito** Authorizer.
3. **VPC Link Tunneling:** Validated requests securely tunnel into the Default VPC (`172.31.0.0/16`) via **VPC Link V2**.
4. **Internal Routing:** Traffic reaches the **Internal Application Load Balancer (ALB)**, which has NO public IP and is strictly private.
5. **EC2 Backend:** The ALB forwards traffic to the **EC2 Instance** running FastAPI (Port 8000). The Security Group is chained so that Port 8000 ONLY accepts traffic from the ALB. Direct internet access is 100% blocked.
6. **Vector Database:** The **Qdrant** database runs via Docker, bound strictly to `127.0.0.1:6333` (localhost loopback), ensuring zero external network exposure.
7. **Cloud Services:** The EC2 backend communicates outbound with **Amazon Bedrock** (Claude Haiku) and **Amazon DynamoDB** for clinical reasoning and session storage.

> [!TIP]
> **Why use the Default VPC instead of a Private Subnet + NAT Gateway?**
> Standard enterprise architecture places backend servers in Private Subnets behind a NAT Gateway. However, a NAT Gateway costs ~$32/month. By deploying in the Default VPC with an Internal ALB and Security Group chaining (blocking all direct inbound IPs except the ALB), we achieve the **exact same Zero-Trust isolation** while reducing unnecessary costs.

---

### Production Cost Summary

This architecture is optimized for a production environment, balancing security, performance, and cost. The estimated monthly cost is **~$72.7/month**.

| Service | Estimated Monthly Cost |
|---|---|
| EC2 t3.micro (Singapore) | ~$9.50 |
| Public IPv4 | $3.65 |
| EBS Storage (30 GiB gp3) | ~$3.00 |
| Application Load Balancer | ~$24.24 |
| API Gateway (VPC Link REST) | $18.25 |
| API Gateway Requests | ~$0.00 |
| **Amazon Bedrock (Claude Haiku)** | **~$14.00** |
| Amazon DynamoDB | <$0.10 |
| Amazon Cognito & CloudWatch | $0.00 |
| Data Transfer | ~$0.05 |
| **TOTAL** | **~$72.7/month** |