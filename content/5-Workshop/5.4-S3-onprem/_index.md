---
title: "Step 2: Private Integration & API Gateway"
date: 2024-01-01
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

In this step, we configure the secure private network topology connecting **Amazon API Gateway** to the **EC2 Backend** using **VPC Link V2** and an **Internal Application Load Balancer (ALB)**.

---

### 1. Create Internal Application Load Balancer (ALB)
1. Open **EC2 Console** -> **Load Balancers** -> **Create Load Balancer**.
2. Select **Application Load Balancer**:
   - **Scheme:** `Internal` (Private VPC subnets only)
   - **IP Address Type:** IPv4
   - **Listeners:** HTTP on Port 80
3. Create a Target Group `pedix-backend-tg`:
   - **Target Type:** Instance
   - **Protocol/Port:** HTTP / 8000
   - **Health Check Path:** `/health`
   - Register `pedix-backend-server` as the target.

---

### 2. Configure VPC Link V2
1. Open **API Gateway Console** -> **VPC Links** -> **Create VPC Link**.
2. Choose **VPC Link for HTTP / REST APIs (V2)**.
3. Select your target VPC and subnets to generate Elastic Network Interfaces (ENIs).

---

### 3. Build API Gateway REST Proxy & SSE Streaming Support
1. Create a REST API named `Pedix-API`.
2. Add a proxy resource `{proxy+}` with `ANY` method:
   - **Integration Type:** VPC Link
   - **VPC Link:** Select your VPC Link V2
   - **Endpoint URL:** `http://<internal-alb-dns>/{proxy}`
3. Enable CORS support and set deployment stage to `prod`.
