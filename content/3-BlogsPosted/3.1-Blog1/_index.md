---
title: "Blog 1: Zero-Trust with API Gateway & VPC Link"
weight: 10
---

# Exposing Private Backends Securely: API Gateway, VPC Link v2, and Internal ALB

**View the published post on AWS Study Group:** [Insert Link Here](https://www.facebook.com/groups/awsstudygroupfcj/...)

*AWS Study Group Sharing – First Cloud AI Journey (FCAJ)*  
*Based on: [Build scalable REST APIs using Amazon API Gateway private integration with Application Load Balancer](https://aws.amazon.com/blogs/compute/build-scalable-rest-apis-using-amazon-api-gateway-private-integration-with-application-load-balancer) – AWS Compute Blog*

---

## 1. The Challenge of Private Backends

When deploying a backend service on AWS-such as the FastAPI + Qdrant AI triage backend we built-security best practices dictate that the compute instance must reside in a private subnet. The objective is strict: **The backend must not have any public inbound rules for application ports**. All external requests must be routed through a controlled, secure entry point.

## 2. Why Choose VPC Link v2 + ALB over NLB?

A classic architectural pattern involves routing traffic like this: `API Gateway -> VPC Link -> Network Load Balancer (NLB) -> Application Load Balancer (ALB) -> Backend`. This approach introduces an extra network hop and a Load Balancer solely to serve as a bridge. Furthermore, because NLB operates at Layer 4, it lacks path-based routing capabilities.

The AWS Compute Blog introduces a modern capability: **API Gateway REST APIs can integrate privately directly into an Internal ALB via VPC Link v2**. This removes the need for an NLB bridge while retaining full Layer 7 routing (such as path rules and HTTP health checks) at the ALB layer. A single VPC Link v2 can also target multiple ALBs or NLBs simultaneously.

![Blog 1 Illustration](/images/blogs/blog1.png)

## 3. The Architecture

```text
CloudFront -> API Gateway (with Cognito Authorizer)
           -> VPC Link v2 
           -> Internal ALB 
           -> Target Group :8000 (Healthy)
           -> EC2 private subnet (FastAPI + Qdrant localhost only)
```

**Key Security Concept:** The security group for the backend instance only allows inbound traffic from the **ALB's Security Group**, not from a broad IP range like `0.0.0.0/0`. This ensures that even if the ALB changes its underlying IP addresses, the security rules remain intact.

## 4. Practical Experience: Resource Shadowing with `{proxy+}`

During my implementation, the standard route (`/api/{proxy+}`, Buffered) worked flawlessly. However, when I added a specific route for SSE streaming-`/api/chat/stream` (Response Transfer Mode = **Stream** to return tokens in real-time)-every other request under `/api/chat/...` (e.g., `/api/chat/message`) suddenly returned 404 errors.

**The Cause:** Creating an explicit sub-resource (like `/api/chat/stream`) prompted API Gateway to implicitly create a parent resource (`/api/chat`). This implicit resource effectively "shadowed" the wildcard `{proxy+}` which was previously handling all other paths under `/api/chat/`.

**The Solution:** I had to add a child `{proxy+}` route directly under the new `/api/chat/` parent, point it to the same VPC Link (Buffered), and redeploy the API stage. As a result, both the streaming route and the standard REST routes function correctly in parallel.

**Lesson learned:** API Gateway explicit resources on a sub-path can break wildcard routes at the parent level, requiring manual synchronization.

## 5. Other Operational Findings

- **Uvicorn State Loss:** Running `uvicorn --workers 2` caused Server-Sent Events (SSE) state loss (`Request ID expired`) because the in-memory request store wasn't shared between processes. I had to revert to `--workers 1`.
- **Connection Drops:** CloudFront and ALB eagerly drop SSE connections if the AI reasoning loop takes too long. I resolved this by reducing the `heartbeat_interval` from 15s down to 5s to keep the connection "alive."

## 6. Conclusion

Building a true zero-trust connection is about the subtle details: properly configuring Security Group sources, selecting the right Response Transfer Mode, and understanding how API Gateway processes parent and child resources when mixing wildcards with explicit paths. For backends that need to be completely hidden in private subnets, the **API Gateway + VPC Link v2 + Internal ALB** pattern is an excellent choice.

---

**Reference:**
- Vijay Menon, Christian Silva – *Build scalable REST APIs using Amazon API Gateway private integration with Application Load Balancer*, AWS Compute Blog. [Link](https://aws.amazon.com/blogs/compute/build-scalable-rest-apis-using-amazon-api-gateway-private-integration-with-application-load-balancer)