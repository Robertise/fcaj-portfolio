---
title: "Proposal"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
---
In this section, we present the detailed technical proposal and deployment execution for the Pedix workshop.

# Pedix - Agentic RAG-Powered Pediatric Health Navigator
## An Age-Stratified Level 4 Agentic System Deployed on AWS

### 1. Executive Summary
**Pedix** is a full-stack, cloud-deployed AI health assistant web application designed specifically for parents and caregivers navigating pediatric health symptoms for children aged **0 to 5 years (under-5)**. Unlike generic symptom checkers or chatbots, Pedix executes an age-stratified, multi-stage Level 4 Agentic Retrieval-Augmented Generation (RAG) reasoning loop powered by Amazon Bedrock (Claude Haiku), Qdrant vector search, and 12 AWS cloud services. It provides evidence-backed care pathway recommendations, urgency classification (based on ESI v4), and pre-visit checklists derived directly from WHO and NICE pediatric guidelines. The system is deployed completely on the AWS cloud in `ap-southeast-1` optimizing for 100% Free Tier limit maximization with a total operating cost of just ~$72.7/month.

### 2. Problem Statement
#### What’s the Problem?
Childhood symptoms (e.g. fever or respiratory distress) require dramatically different clinical responses depending on exact age: a 38°C fever in a 2-week-old newborn is a medical emergency requiring immediate pediatric ER evaluation, whereas the same temperature in a 3-year-old toddler is often manageable at home with monitoring. Existing symptom checkers often rely on adult-adapted models or static pipelines that fail to apply age-stratified clinical thresholds, creating either dangerous under-triage or unnecessary emergency room visits.

#### The Solution
Pedix introduces an **Age-Stratified Level 4 Agentic RAG** architecture. Before invoking the LLM, a deterministic pediatric safety screen (Stage 0) scans for life-threatening red flags in <10ms. Once safe, the system resolves the child's age first (Stage 1), applies strict `age_group` pre-filtering on the Qdrant vector database (Stage 2), reasons over retrieved evidence using the ESI v4 framework (Stage 3), self-evaluates completeness (Stage 4 reflection loop), and generates warm, parent-facing recommendations with cited sources (Stage 5).

#### Benefits and Return on Investment
- **Clinical Safety & Transparency:** Visible 5-stage reasoning trace and cited WHO/NICE guideline chunks prevent black-box AI recommendations.
- **Cost Governance on AWS:** Optimised production deployment using EC2 (t3.micro), Qdrant Docker container, DynamoDB On-Demand, and API Gateway VPC Link costing **~$72.7/month**, well within budget limits.
- **Scalability & Maintenance:** Dynamic knowledge base indexing allows administrators to upload updated pediatric guidelines without redeploying backend infrastructure.

### 3. Comprehensive Solution Architecture
Pedix utilizes a strict Zero-Trust production AWS architecture hosted in `ap-southeast-1` (Singapore):

![Pedix AWS Cloud Architecture](/images/2-Proposal/pedix_architecture.png)

#### Zero-Trust Network Isolation & Core Services
- **Amazon S3 + CloudFront:** Hosts the React (Vite) frontend globally via HTTPS. The S3 bucket (`pedix-frontend-prod`) is strictly private (Block All Public Access) and served via CloudFront Origin Access Control (OAC).
- **Amazon API Gateway (Regional REST):** Validates tokens via Cognito JWT Authorizer, tunneling into the VPC via VPC Link V2 (`fzvy02`). It natively supports Server-Sent Events (SSE) streaming (`/api/chat/stream`) for real-time trace generation.
- **Application Load Balancer (Internal ALB):** Configured as an internal-only scheme (`pedix-internal-alb`), it bridges the public API Gateway to the private EC2 backend.
- **Amazon EC2 (t3.micro + 2GB Swap):** Hosts the FastAPI backend server on a private IP (`172.31.42.140`). Security group chaining (`pedix-ec2-sg`) ensures it only accepts inbound traffic from the ALB (`pedix-alb-sg`), completely blocking direct internet access on port 8000.
- **Qdrant Vector DB (Docker v1.10.1):** Stores 384-dimensional dense vectors mapped to an EBS gp3 volume. It is strictly bound to the localhost interface (`127.0.0.1:6333`) to prevent external network exposure.
- **Amazon Bedrock (Claude Haiku):** Powers all structured query analysis, clinical reasoning, reflection, and response generation via `global.anthropic.claude-haiku-4-5-20251001-v1:0`.
- **Amazon DynamoDB:** Pay-per-request tables store sessions (`pedix_sessions`), child profiles (`pedix_profiles`), and system analytics logs (`pedix_analytics_log`).
- **Amazon Cognito + AWS Lambda:** Manages user authentication (`ap-southeast-1_Osm01gaEp`) with a Post-Confirmation Lambda trigger assigning users to the `pedix-users` group.

### 4. Technical Implementation & Deployment Details
#### Implementation Phases
1. **Research & Domain Modeling (Month 1):** Curated pediatric guidelines from WHO and NICE CG160. Formulated ESI v4 urgency mapping and age-stratified pre-filters (`newborn`, `young_infant`, `infant`, `toddler`, `preschool`).
2. **Architecture & Cost Optimization (Month 2):** Designed a custom Python agent orchestrator. Replaced OpenSearch Serverless with Qdrant Docker on EC2 to eliminate ~$60/month base costs. Configured a 2GB EBS Swap to prevent EC2 OOM issues during heavy tensor embeddings.
3. **Backend & SSE Streaming (Month 2-3):** Built a multi-stage retrieval pipeline. Resolved CloudFront/ALB idle connection timeouts by implementing a 5.0-second SSE heartbeat (`_call_with_heartbeat`) during long reasoning loops. Set FastAPI workers to `--workers 1` to ensure shared `PendingRequestStore` state across SSE requests.
4. **Zero-Trust Deployment (Month 3):** Deployed infrastructure via AWS Console/CLI. Configured Security Group chaining and resolved an API Gateway proxy shadowing issue by mapping `/api/chat/{proxy+}` correctly to the VPC Link.

### 5. Timeline & Milestones
- **Month 1 (May 2026):** Requirements gathering, WHO/NICE knowledge base curation, and Stage 0 safety screen development.
- **Month 2 (June 2026):** FastAPI backend, Qdrant integration, Bedrock tool-use prompts, DynamoDB schema design, and React UI construction.
- **Month 3 (July & August 2026):** Cloud deployment (Cognito, VPC Link, ALB, CloudFront), SSE heartbeat keep-alive optimization, Zero-Trust network segmentation, and complete end-to-end verification.

### 6. Production Cost Breakdown (~100 users)
The production environment is heavily optimized for AWS Free Tier in `ap-southeast-1`:

| Service | Configuration | Est. Monthly Cost |
|---|---|---|
| EC2 (t3.micro) | Linux Ubuntu 2 GiB RAM + 2 GiB Swap | ~$9.50 |
| Public IPv4 | EC2 assigned IP | $3.65 |
| EBS Volume | 30 GiB gp3 storage | ~$3.00 |
| Internal ALB | HTTP listener + target group | ~$24.24 |
| VPC Link V2 | REST API Private Integration | $18.25 |
| Bedrock Inference | Claude Haiku (~800 queries) | ~$14.00 |
| DynamoDB | Pay-per-request (4 tables under 25GB) | <$0.10 |
| CloudFront & S3 | Static assets & CDN transfer | ~$0.05 |
| Cognito & CloudWatch | Free Tier limits | $0.00 |
| **Total Estimated Cost** | | **~$72.7 / month** |

### 7. Risk Assessment & Mitigations
- **Clinical Safety Misses:** Mitigated by deterministic Stage 0 screening (<10ms) that routes high-risk cases (e.g., cyanosis, bulging fontanelle, fever under 90 days) directly to emergency advice without waiting for LLM loops.
- **ALB / API Gateway SSE Timeouts:** Resolved by implementing 5-second heartbeat SSE events to prevent network components from dropping the connection while Bedrock processes long reasoning traces.
- **Security & Authorization Leaks:** Mitigated by API Gateway Cognito JWT Authorizers, secondary FastAPI JWKS key validation, and ALB Security Group chaining that prevents direct external access to backend endpoints.

### 8. Expected Outcomes
- **Demonstrable Level 4 Agentic RAG:** A fully functional, age-aware pediatric health assistant deployed live on AWS.
- **Transparent Healthcare AI:** A collapsible reasoning trace allowing parents to inspect every decision stage and cited source in real-time via SSE.
- **Comprehensive Cloud Architecture:** Production deployment combining 12 AWS services with strict DevSecOps, zero-trust network boundaries, and rigorous cost governance.