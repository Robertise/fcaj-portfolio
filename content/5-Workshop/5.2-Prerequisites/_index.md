---
title: "2. Prerequisites & IAM Roles"
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

# AWS Prerequisites & IAM Roles Setup

Before provisioning the core infrastructure, we must configure our AWS Region, enable access to Amazon Bedrock AI models, and create the necessary Identity and Access Management (IAM) Roles to ensure our components can communicate securely without hardcoded credentials.

---

### Step 1: AWS Region Selection

We will deploy all resources in the **Singapore** region.
1. Log in to the [AWS Management Console](https://console.aws.amazon.com/).
2. In the top-right corner, click the Region dropdown menu.
3. Select **Asia Pacific (Singapore) `ap-southeast-1`**.

> [!WARNING]
> It is critical to stay in `ap-southeast-1` for the duration of this workshop. AWS resources (like VPCs and EC2 instances) are region-specific.

---

### Step 2: Request Amazon Bedrock Model Access

Pedix requires access to Anthropic's Claude 3 Haiku model for clinical reasoning and document chunking.

1. In the AWS Console search bar, type **Bedrock** and open it.
2. In the left navigation pane, scroll down and click on **Model access** (under the Bedrock settings).
3. Click the orange **Enable specific models** button (or **Manage model access**).
4. Check the box next to **Anthropic** > **Claude 3 Haiku**.
5. Scroll to the bottom and click **Save changes**. 
6. Wait a few minutes. The access status should change to **Access granted**.

---

### Step 3: Create the `Pedix-EC2-Role` (For Backend Server)

To follow the AWS Well-Architected Framework's security pillar, we will *never* store AWS Access Keys in our EC2 `.env` file. Instead, we attach an IAM Role to the EC2 instance.

1. Search for **IAM** in the AWS Console and open it.
2. Click **Roles** on the left menu, then click **Create role**.
3. Under **Trusted entity type**, select **AWS service**.
4. Under **Use case**, choose **EC2**, then click **Next**.
5. In the permissions search bar, search for and select the checkbox for:
   * `AmazonSSMManagedInstanceCore` *(This allows us to securely SSH into the server via Session Manager without opening port 22).*
6. Click **Next**, name the role **`Pedix-EC2-Role`**, and click **Create role**.

#### Attach the Custom Inline Policy
We need to grant this role access to Bedrock and DynamoDB.
1. Click on your newly created **`Pedix-EC2-Role`**.
2. Under the **Permissions** tab, click **Add permissions** > **Create inline policy**.
3. Switch to the **JSON** tab and paste the following policy:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "BedrockAccess",
            "Effect": "Allow",
            "Action": [
                "bedrock:InvokeModel",
                "bedrock:InvokeModelWithResponseStream"
            ],
            "Resource": "*"
        },
        {
            "Sid": "DynamoDBListTables",
            "Effect": "Allow",
            "Action": [
                "dynamodb:ListTables"
            ],
            "Resource": "*"
        },
        {
            "Sid": "DynamoDBAccess",
            "Effect": "Allow",
            "Action": [
                "dynamodb:GetItem",
                "dynamodb:PutItem",
                "dynamodb:UpdateItem",
                "dynamodb:DeleteItem",
                "dynamodb:Query",
                "dynamodb:Scan"
            ],
            "Resource": [
                "arn:aws:dynamodb:*:*:table/pedix_*",
                "arn:aws:dynamodb:*:*:table/pedix_*/index/*"
            ]
        }
    ]
}
```
4. Click **Next**, name the policy **`Pedix-EC2-Permissions`**, and click **Create policy**.

---

### Step 4: Create the `Pedix-PostConfirmation-Role` (For AWS Lambda)

This role will be used by our AWS Lambda function to automatically add newly registered users to a Cognito group.

1. Go back to IAM > **Roles** > **Create role**.
2. Under **Trusted entity type**, select **AWS service**.
3. Under **Use case**, select **Lambda**, then click **Next**.
4. Search for and check **`AWSLambdaBasicExecutionRole`** *(allows Lambda to write logs to CloudWatch)*.
5. Click **Next**, name it **`Pedix-PostConfirmation-Role`**, and click **Create role**.

#### Attach the Custom Inline Policy
1. Click on **`Pedix-PostConfirmation-Role`**.
2. Click **Add permissions** > **Create inline policy**.
3. Switch to the **JSON** tab and paste:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "cognito-idp:AdminAddUserToGroup"
            ],
            "Resource": "*"
        }
    ]
}
```
4. Click **Next**, name the policy **`Pedix-Cognito-GroupAssignment`**, and click **Create policy**.

You are now ready to deploy the actual cloud infrastructure!