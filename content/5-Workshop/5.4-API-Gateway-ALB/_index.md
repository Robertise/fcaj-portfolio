---
title: "4. Internal ALB & API Gateway"
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

# API Gateway, VPC Link V2 & Internal ALB Configuration

Right now, your EC2 backend is completely hidden from the internet. We will now build the secure bridge (VPC Link) that allows the public API Gateway to talk to your private backend using an Internal Load Balancer.

---

### Step 1: Create the ALB Security Group

1. In the EC2 Dashboard, go to **Security Groups** and click **Create security group**.
2. **Name:** `pedix-alb-sg`.
3. **Description:** Security group for Internal ALB.
4. **VPC:** Select the Default VPC.
5. **Inbound Rules:** Add a rule: Type `HTTP` (Port 80), Source `Anywhere-IPv4` (`0.0.0.0/0`).
6. Click **Create security group**.

> [!CAUTION]
> **Security Group Chaining:** You must now go back to your `pedix-ec2-sg` (the EC2's security group), edit the Inbound Rules, and change the Port 8000 rule so its Source is `pedix-alb-sg`. This ensures your EC2 instance will *only* accept traffic from the Load Balancer!

---

### Step 2: Create the Target Group

1. In the EC2 Dashboard, scroll down on the left menu to **Target Groups** and click **Create target group**.
2. **Target type:** Select **Instances**.
3. **Target group name:** `pedix-backend-tg`.
4. **Protocol:** `HTTP`, **Port:** `8000`.
5. **VPC:** Select your Default VPC.
6. **Health checks:** Set Health check path to `/api/health`.
7. Click **Next**.
8. Select your `Pedix-Backend-Server` instance, click **Include as pending below**, and click **Create target group**.

---

### Step 3: Create the Internal Application Load Balancer

1. In the EC2 menu, click **Load Balancers** > **Create load balancer**.
2. Click **Create** under **Application Load Balancer**.
3. **Load balancer name:** `pedix-internal-alb`.
4. **Scheme:** Select **Internal** (Crucial! Do not select Internet-facing).
5. **Network mapping:** Select all Availability Zones (`ap-southeast-1a`, `1b`, `1c`).
6. **Security groups:** Select `pedix-alb-sg`.
7. **Listeners and routing:** Protocol `HTTP`, Port `80`, Forward to `pedix-backend-tg`.
8. Click **Create load balancer**.

---

### Step 4: Create VPC Link V2 (API Gateway to VPC)

1. Search for **API Gateway** in the AWS Console.
2. In the left menu, click **VPC links** > **Create**.
3. **VPC link version:** Select **VPC link for REST APIs**.
4. **Name:** `pedix-vpclink`.
5. **Target Load Balancer:** Select `pedix-internal-alb`.
6. Click **Create**. It takes 2-3 minutes for the status to change to *Available*.

---

### Step 5: Configure API Gateway & SSE Streaming

1. In the API Gateway console, click **APIs** > **Create API**.
2. Find **REST API** (the one *without* Private) and click **Build**.
3. **API name:** `Pedix-API`. Endpoint Type: `Regional`. Click **Create API**.

#### Create the `{proxy+}` Catch-All Route
1. Click **Create resource**. Turn on **Proxy resource**, leave Resource path as `/{proxy+}`. Click **Create resource**.
2. Select the `ANY` method that was just created under `/{proxy+}`.
3. **Integration type:** Select **VPC Link**.
4. Turn on **Use proxy integration**.
5. **Method:** `ANY`.
6. **VPC Link:** Select `pedix-vpclink`.
7. **Endpoint URL:** `http://internal-pedix-internal-alb-[id].ap-southeast-1.elb.amazonaws.com/api/{proxy}` *(Replace the ALB DNS name with your actual ALB DNS name from the EC2 console)*.
8. Click **Save**.

#### Create the Real-time SSE Stream Route
Server-Sent Events (SSE) require API Gateway to leave the connection open, bypassing the normal response buffer.
1. Click the root `/` path, then click **Create resource**.
2. Resource Name: `api`, click Create.
3. Select `/api`, click **Create resource**. Resource Name: `chat`, click Create.
4. Select `/api/chat`, click **Create resource**. Resource Name: `stream`, click Create.
5. Select `/api/chat/stream`, click **Create method**.
6. **Method type:** `GET`.
7. **Integration type:** Select **VPC Link**.
8. Turn on **Use proxy integration**.
9. **Endpoint URL:** `http://internal-[YOUR-ALB-DNS]/api/chat/stream`.
10. Click **Create**.

> [!IMPORTANT]
> To enable streaming, you must go to the **Integration Request** settings for the `GET /api/chat/stream` method, and change the **Response Transfer Mode** to **Stream** instead of **Buffered**.

#### Create the Chat Sub-path Proxy (Fixing the 404 Error)
Because we created a specific route for `/api/chat/stream`, API Gateway automatically created the parent folder `/api/chat`. This unintentionally blocks the top-level `/{proxy+}` from routing other chat endpoints like `/api/chat/message` or `/api/chat/session` (they will return a 404 Not Found).
1. Select the `/api/chat` resource and click **Create resource**.
2. Turn on **Proxy resource**, leave Resource path as `/{proxy+}`. Click **Create resource**.
3. Select the `ANY` method under `/api/chat/{proxy+}`.
4. **Integration type:** Select **VPC Link**.
5. Turn on **Use proxy integration**.
6. **VPC Link:** Select `pedix-vpclink`.
7. **Endpoint URL:** `http://internal-[YOUR-ALB-DNS]/api/chat/{proxy}`.
8. Click **Save**.

#### Deploy the API
1. Click the orange **Deploy API** button.
2. Select **New stage**, name it `prod`, and click Deploy.
3. You will receive an Invoke URL (e.g., `https://96hnl890q4.execute-api.ap-southeast-1.amazonaws.com/prod`). 
4. Test it by opening `https://[YOUR_API_ID].execute-api.ap-southeast-1.amazonaws.com/prod/api/health` in your browser. You should see `{"status":"ok","service":"pedix-backend"}`!
