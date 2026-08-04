---
title: "Prerequisites"
date: 2024-01-01 
weight: 2 
chapter: false
pre: " <b> 5.2. </b> "
---

Before deploying **Pedix** on AWS, ensure you have configured the following accounts, services, and local tools.

---

### 1. AWS Account & Region Setup
- **AWS Region:** Select `ap-southeast-1` (Singapore) for optimal latency and Amazon Bedrock model availability.
- **AWS IAM Administrator Access:** Required to create IAM Roles, DynamoDB tables, VPC Links, and CloudFront distributions.

---

### 2. Amazon Bedrock Model Access
1. Navigate to **Amazon Bedrock Console** -> **Model Access**.
2. Request access for the following models:
   - **Anthropic Claude 3.5 Sonnet** (for clinical reasoning & reflection)
   - **Anthropic Claude 3 Haiku** (for fast query analysis & safety screening)

---

### 3. IAM Role Configuration (`Pedix-EC2-Role`)
Create an IAM Role for the EC2 instance with the following attached managed policies:
- `AmazonBedrockFullAccess`
- `AmazonDynamoDBFullAccess`
- `CloudWatchLogsFullAccess`

---

### 4. Local Development Tools
- **AWS CLI v2:** Installed and authenticated (`aws configure`).
- **Python 3.11+:** Installed locally for scripting and ingestion testing.
- **Docker Desktop:** Installed for running local Qdrant containers during testing.