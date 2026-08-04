---
title: "Bước 3: Xác thực Cognito & Bảng DynamoDB"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

Trong bước này, chúng ta cấu hình hệ thống xác thực người dùng qua **Amazon Cognito**, tự động hóa phân quyền bằng **AWS Lambda Trigger**, và khởi tạo 4 bảng dữ liệu **Amazon DynamoDB**.

---

### 1. Khởi tạo Amazon Cognito User Pool
1. Truy cập **Amazon Cognito Console** -> **Create User Pool**.
2. Cấu hình đăng nhập: Email & Username.
3. Chính sách mật khẩu: Tối thiểu 8 ký tự.
4. Tạo App Client `pedix-react-client` (chế độ Public client cho SPA React).

---

### 2. Triển khai AWS Lambda Post-Confirmation Trigger
Khởi tạo Lambda function `pedix-post-confirm-trigger` (Python 3.11):
```python
import boto3

cognito = boto3.client('cognito-idp')

def lambda_handler(event, context):
    user_pool_id = event['userPoolId']
    user_name = event['userName']
    
    # Tự động gán người dùng mới kích hoạt vào nhóm 'pedix-users'
    cognito.admin_add_user_to_group(
        UserPoolId=user_pool_id,
        Username=user_name,
        GroupName='pedix-users'
    )
    return event
```
Gán hàm Lambda này vào sự kiện **Post Confirmation Trigger** trong Cognito User Pool.

---

### 3. Tạo 4 Bảng Amazon DynamoDB
Tạo 4 bảng dữ liệu chế độ On-Demand (`PAY_PER_REQUEST`):
1. **`pedix_sessions`**: Primary Key `session_id` (String) — Lưu trữ lịch sử hội thoại.
2. **`pedix_profiles`**: Primary Key `child_id` (String), GSI `user_id` — Lưu thông tin độ tuổi, giới tính và tiền sử trẻ.
3. **`pedix_analytics_log`**: Primary Key `log_id` (String), Sort Key `timestamp` — Nhật ký phân tích mức độ ESI & thời gian phản hồi.
4. **`pedix_documents`**: Primary Key `doc_id` (String) — Danh mục tài liệu y khoa WHO & NICE.
