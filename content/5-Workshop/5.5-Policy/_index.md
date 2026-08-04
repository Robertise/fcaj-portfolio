---
title: "Step 3: Cognito Auth & DynamoDB Tables"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

In this step, we configure user authentication using **Amazon Cognito**, deploy an **AWS Lambda Post-Confirmation trigger**, and provision 4 **Amazon DynamoDB** tables for persistent storage.

---

### 1. Amazon Cognito User Pool & App Client
1. Open **Amazon Cognito Console** -> **Create User Pool**.
2. Configure Sign-in Options: Email & Username.
3. Set Password Policy: Minimum 8 characters.
4. Create an App Client `pedix-react-client` (no client secret required for public React SPAs).

---

### 2. Deploy AWS Lambda Post-Confirmation Trigger
Create an AWS Lambda function `pedix-post-confirm-trigger` (Python 3.11):
```python
import boto3

cognito = boto3.client('cognito-idp')

def lambda_handler(event, context):
    user_pool_id = event['userPoolId']
    user_name = event['userName']
    
    # Automatically assign newly confirmed users to default 'pedix-users' group
    cognito.admin_add_user_to_group(
        UserPoolId=user_pool_id,
        Username=user_name,
        GroupName='pedix-users'
    )
    return event
```
Attach this Lambda function as the **Post Confirmation Trigger** on the Cognito User Pool.

---

### 3. Provision Amazon DynamoDB Tables
Create 4 DynamoDB tables using On-Demand capacity mode (`PAY_PER_REQUEST`):
1. **`pedix_sessions`**: Primary Key `session_id` (String) — Stores chat session metadata & history.
2. **`pedix_profiles`**: Primary Key `child_id` (String), GSI `user_id` — Stores age, gender, and clinical history profiles.
3. **`pedix_analytics_log`**: Primary Key `log_id` (String), Sort Key `timestamp` — Audit logging for ESI triage levels & latency metrics.
4. **`pedix_documents`**: Primary Key `doc_id` (String) — Registry of uploaded pediatric WHO & NICE guideline PDF files.
