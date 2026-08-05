---
title: "6. Frontend S3 & CloudFront"
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

# Frontend S3/CloudFront Deployment & System Validation

In this final step, we will compile the React frontend and deploy it securely to a private Amazon S3 bucket, served globally via Amazon CloudFront CDN.

---

### Step 1: Build the React Application

1. On your local machine (not the EC2), open the `frontend` folder of the Pedix repository.
2. Create a file named `.env.production` in the root of the `frontend` folder.
3. Paste the following variables, replacing the placeholders with your actual Cognito IDs and API Gateway Invoke URL:

```env
VITE_API_BASE_URL=https://[YOUR_API_ID].execute-api.ap-southeast-1.amazonaws.com/prod
VITE_COGNITO_USER_POOL_ID=ap-southeast-1_[YOUR_POOL]
VITE_COGNITO_CLIENT_ID=[YOUR_CLIENT_ID]
VITE_COGNITO_REGION=ap-southeast-1
```

> [!WARNING]
> **UTF-8 Encoding Warning (Windows PowerShell):** If you create this file using PowerShell's `echo` or `Out-File`, it defaults to UTF-16LE encoding. Vite will silently ignore the file, causing your login/register buttons to silently fail in production. Ensure the file is saved as **UTF-8**.

4. Run the build command:
```bash
npm install
npm run build
```
This will generate a `dist` folder containing your compiled, production-ready static assets.

---

### Step 2: Create a Private Amazon S3 Bucket

1. Go to the **Amazon S3** console and click **Create bucket**.
2. **Bucket name:** `pedix-frontend-prod-[your-initials]` (Must be globally unique).
3. **Region:** `ap-southeast-1` (Singapore).
4. **Block Public Access settings:** Ensure **Block all public access** is **CHECKED** (We want the bucket completely private).
5. Click **Create bucket**.
6. Open your new bucket, click **Upload**, drag and drop all files and folders from your `dist` directory, and click **Upload**.

---

### Step 3: Create the Amazon CloudFront Distribution

1. Go to the **CloudFront** console and click **Create Distribution**.
2. **Origin domain:** Select your S3 bucket from the dropdown.
3. **Origin access:** Select **Origin access control settings (recommended)**. 
   * Click **Create control setting** and accept the defaults.
   * **IMPORTANT:** CloudFront will remind you to update your S3 bucket policy. Click **Copy policy**. We will do this right after creating the distribution.
4. **Default cache behavior:**
   * Viewer protocol policy: **Redirect HTTP to HTTPS**.
5. **Web Application Firewall (WAF):** Select **Do not enable security protections** (to avoid extra costs).
6. Click **Create distribution**.

#### Update S3 Bucket Policy (OAC)
1. Go back to your S3 bucket > **Permissions** tab.
2. Scroll to **Bucket policy** and click **Edit**.
3. Paste the policy you copied from CloudFront. It allows CloudFront to read the private bucket. Click **Save changes**.

---

### Step 4: Configure SPA Fallback Error Pages

Since React is a Single Page Application (SPA), all routing is handled by the browser. If a user visits `/chat` directly, CloudFront will look for a file named `chat` in S3 and return a 403 Forbidden error.

1. Open your CloudFront Distribution and go to the **Error pages** tab.
2. Click **Create custom error response**.
3. HTTP error code: `403: Forbidden`.
4. Customize error response: **Yes**.
5. Response page path: `/index.html`.
6. HTTP Response code: `200: OK`.
7. Click **Create custom error response**.
8. Repeat the exact same steps for HTTP error code `404: Not Found`.

---

### Step 5: System Validation & Handover

Your architecture is now 100% complete!

Wait a few minutes for the CloudFront distribution to finish deploying. Copy the **Distribution domain name** (e.g., `https://d2bx3usq72976a.cloudfront.net`) and open it in your browser.

**End-to-End Checklist:**
- [x] Website loads securely over HTTPS via CloudFront.
- [x] You can register a new account and receive a verification email.
- [x] Logging in works seamlessly (Vite `.env.production` is correct).
- [x] Chat messages process instantly via SSE Streaming from the VPC-Linked Backend.

> [!TIP]
> **Cache Invalidation:** Whenever you update your frontend code and re-upload the `dist` folder to S3, you must go to CloudFront > **Invalidations** > **Create invalidation** > enter `/*` and run it. This forces edge locations to fetch your new code.

**Congratulations on deploying a production-grade Agentic RAG system on AWS!**