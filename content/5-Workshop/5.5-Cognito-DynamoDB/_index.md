---
title: "5. Cognito Auth & DynamoDB"
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

# Cognito Auth, Lambda Triggers & DynamoDB

Pedix uses Amazon Cognito for secure user authentication (Free for up to 50,000 MAUs) and Amazon DynamoDB for serverless NoSQL storage.

---

### Step 1: Create the Cognito User Pool

1. Go to the **Amazon Cognito** console and click **Create user pool**.
2. **Provider types:** Select **Cognito user pool**.
3. **Cognito user pool sign-in options:** Check **Email**. Click Next.
4. **Password policy:** Leave defaults (Cognito defaults).
5. **Multi-factor authentication:** Select **No MFA** (for simplicity in this workshop). Click Next.
6. **Required attributes:** Check `name`. Click Next.
7. **Email provider:** Select **Send email with Cognito** (This is free). Click Next.
8. **User pool name:** Enter `pedix-user-pool`.
9. **Initial app client:** Check **Public client**, name it `pedix-app-client`, and select **Don't generate a client secret** (Since our frontend is a React SPA, secrets cannot be stored securely).
10. Review and click **Create user pool**.

> [!NOTE]
> Open your new user pool. Copy the **User pool ID** and the **Client ID** (from the App Integration tab). You must now SSH into your EC2 instance and update `/home/ubuntu/pedix/.env` with these actual values, replacing the `PLACEHOLDER` text! Restart the backend with `sudo systemctl restart pedix-backend`.

#### Create User Groups
1. Inside your User Pool, go to the **Groups** tab.
2. Click **Create group**. Name it `pedix-users`.
3. Create another group named `pedix-admins`.

---

### Step 2: Create the Lambda Post-Confirmation Trigger

When a user signs up, we want to automatically assign them to the `pedix-users` group.

1. Go to the **AWS Lambda** console and click **Create function**.
2. Select **Author from scratch**.
3. **Function name:** `Pedix-PostConfirmation`.
4. **Runtime:** Python 3.12.
5. **Execution role:** Select **Use an existing role** and choose `Pedix-PostConfirmation-Role`.
6. Click **Create function**.
7. In the Code source editor, paste the following Python code:

```python
import boto3
import os

def lambda_handler(event, context):
    client = boto3.client('cognito-idp')
    user_pool_id = event['userPoolId']
    username = event['userName']
    group_name = 'pedix-users'
    
    try:
        client.admin_add_user_to_group(
            UserPoolId=user_pool_id,
            Username=username,
            GroupName=group_name
        )
    except Exception as e:
        print(f"Error assigning user to group: {e}")
        raise e
        
    return event
```
8. Click **Deploy**.
9. **Attach to Cognito:** Go back to your Cognito User Pool > **User pool properties** > **Lambda triggers** > **Add Lambda trigger**. Select **Post confirmation** and choose your `Pedix-PostConfirmation` function.

---

### Step 3: DynamoDB Tables

Pedix uses **On-Demand** capacity mode for DynamoDB, meaning you pay exactly $0 if there are no requests.

Actually, you don't need to manually create the tables! When you start the FastAPI backend on your EC2 instance, the application lifecycle event in `main.py` uses `boto3` to automatically create the tables if they don't exist.

To verify:
1. Go to the **DynamoDB** console.
2. Click **Tables** on the left.
3. You should see 4 tables automatically created:
   * `pedix_sessions` (Conversations)
   * `pedix_profiles` (Children Health Profiles)
   * `pedix_documents` (Knowledge Base Registry)
   * `pedix_analytics_log` (Usage metrics)

---

### Step 4: Secure the API Gateway with Cognito

Now we force API Gateway to validate the Cognito JWT token before it even reaches the EC2 backend.

1. Go back to your `Pedix-API` in the **API Gateway** console.
2. On the left menu, click **Authorizers** > **Create New Authorizer**.
3. **Name:** `PedixCognitoAuthorizer`.
4. **Type:** Cognito.
5. **Cognito User Pool:** Select `pedix-user-pool`.
6. **Token Source:** Type `Authorization`.
7. Click **Create**.
8. Go back to **Resources**. Click on the `ANY` method under `/{proxy+}`.
9. Click **Method Request**.
10. Edit the **Authorization** dropdown and select `PedixCognitoAuthorizer` (You may need to refresh the page if it doesn't appear).
11. Click the checkmark to save.
12. **Important:** Click **Deploy API** again to push these security changes to the `prod` stage!
