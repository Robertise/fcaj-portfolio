---
title: "Blog 3"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.3. </b> "
---
{{% notice warning %}}
⚠️ **Note:** The information below is for reference purposes only. Please **do not copy verbatim** for your report, including this warning.
{{% /notice %}}

# SECURING PRIVATE BACKENDS WITH API GATEWAY VPC LINK V2 & INTERNAL ALB

### Architecture Overview
In cloud application security, backend EC2 instances should never be directly exposed to the public internet with open HTTP ports. This article outlines the DevSecOps architecture implemented for **Pedix**, utilizing **Amazon API Gateway REST API**, **VPC Link V2 (HTTP/REST Private Integration)**, and an **Internal Application Load Balancer (ALB)** to establish a zero-trust network perimeter.

---

### Implementation & Traffic Flow

#### 1. Public Entry & Rate Limiting
Client requests hit Amazon API Gateway via HTTPS. API Gateway enforces CORS policies, rate limiting (throttling at 100 requests/sec), and Cognito User Pool JWT token validation.

#### 2. Private Passage via VPC Link V2
Instead of routing over public IP addresses, API Gateway bridges into the private VPC subnet using **VPC Link V2**. This creates ENIs (Elastic Network Interfaces) directly inside the VPC that tunnel API requests securely.

#### 3. Internal Load Balancing & Security Groups
The VPC Link forwards traffic to an **Internal Application Load Balancer (ALB)**. The ALB Security Group is restricted to accept traffic *only* from the VPC Link subnet CIDR. In turn, the EC2 instance Security Group accepts incoming HTTP traffic *only* on port 8000 from the Internal ALB Security Group.

```
[Client] ---> (CloudFront / HTTPS) ---> [API Gateway] 
                                             |
                                     (VPC Link V2)
                                             |
                                  [Internal ALB (Port 80)]
                                             |
                                  [EC2 FastAPI (Port 8000)]
```

#### 4. Server-Sent Events (SSE) Streaming Keep-Alive
Because Bedrock LLM reasoning loops can take 5-15 seconds, default HTTP gateway timeouts (29s) or ALB idle timeouts can trigger dropped connections. The FastAPI backend sends periodic heartbeat frames (`: ping\n\n`) every 5 seconds to keep the VPC Link stream open and fluid for live frontend trace rendering.