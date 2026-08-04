---
title: "Step 4: Frontend Delivery, Testing & Clean-up"
date: 2024-01-01
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

In this final step, we deploy the **React (Vite)** frontend to **Amazon S3** & **CloudFront (OAC)**, perform end-to-end SSE validation, configure cost monitoring, and provide clean-up instructions to prevent unwanted AWS charges.

---

### 1. Frontend Build & CloudFront Distribution
1. Build the React application locally:
   ```bash
   npm run build
   ```
2. Create an Amazon S3 Bucket `pedix-frontend-static-singapore` and sync build assets:
   ```bash
   aws s3 sync dist/ s3://pedix-frontend-static-singapore --delete
   ```
3. Create a **CloudFront Distribution**:
   - **Origin:** S3 Bucket Origin Access Control (OAC).
   - **Viewer Protocol Policy:** Redirect HTTP to HTTPS.
   - **Custom Error Response:** 403/404 -> `/index.html` (SPA routing).

---

### 2. End-to-End Testing & SSE Validation
1. Open the CloudFront domain URL in your browser.
2. Sign up and authenticate via Cognito.
3. Submit a test case: *"3-month-old infant with 38.5°C fever and mild cough."*
4. Verify real-time SSE reasoning trace streaming in the UI:
   - **Stage 0:** Safety Screen Passed (<10ms)
   - **Stage 1:** Age mapped to `young_infant` (90 days)
   - **Stage 2:** Vector Search with `age_group: young_infant` filter
   - **Stage 3:** ESI v4 Classification: Level 2 (Urgent — immediate pediatric review recommended)
   - **Stage 4:** Reflection Loop validated
   - **Stage 5:** Recommendations generated with cited WHO guidelines.

---

### 3. AWS Budgets Alarm & CloudWatch Monitoring
- **AWS Budgets:** Configured email alarm at **$50.00** threshold.
- **CloudWatch Logs:** Log group `/aws/ec2/pedix-fastapi` retention set to 14 days.

---

### 4. Resource Clean-up Instructions
To avoid incurring future cloud costs when teardown is required:
1. **S3 Bucket:** Empty and delete `pedix-frontend-static-singapore`.
2. **CloudFront:** Disable and delete the distribution.
3. **API Gateway & VPC Link:** Delete `Pedix-API` REST API and VPC Link V2.
4. **Internal ALB & Target Group:** Delete ALB and `pedix-backend-tg`.
5. **EC2 Instance:** Terminate `pedix-backend-server` (releases EBS volume).
6. **DynamoDB Tables:** Delete `pedix_sessions`, `pedix_profiles`, `pedix_analytics_log`, and `pedix_documents`.
7. **Cognito User Pool:** Delete `pedix-user-pool`.