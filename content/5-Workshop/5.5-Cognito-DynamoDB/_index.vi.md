---
title: "5. Thiết lập Cognito & DynamoDB"
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

# Xác thực Cognito, Lambda Triggers & DynamoDB

Pedix sử dụng Amazon Cognito để xác thực người dùng an toàn (Miễn phí tới 50.000 MAU) và Amazon DynamoDB để lưu trữ dữ liệu NoSQL Serverless.

---

### Bước 1: Tạo Cognito User Pool

1. Vào bảng điều khiển **Amazon Cognito** và bấm **Create user pool**.
2. **Provider types:** Chọn **Cognito user pool**.
3. **Cognito user pool sign-in options:** Tích vào **Email**. Bấm Next.
4. **Password policy:** Để nguyên mặc định.
5. **Multi-factor authentication:** Chọn **No MFA** (Để đơn giản cho bài thực hành). Bấm Next.
6. **Required attributes:** Tích chọn thuộc tính `name`. Bấm Next.
7. **Email provider:** Chọn **Send email with Cognito** (Tính năng này miễn phí). Bấm Next.
8. **User pool name:** Đặt tên `pedix-user-pool`.
9. **Initial app client:** Tích vào **Public client**, đặt tên `pedix-app-client`, và chọn **Don't generate a client secret** (Vì frontend React không thể bảo mật được secret key).
10. Xem lại cấu hình và bấm **Create user pool**.

> [!NOTE]
> Hãy mở User Pool vừa tạo. Copy **User pool ID** và **Client ID** (nằm ở tab App Integration). Bây giờ bạn phải SSH vào lại EC2 và sửa file `/home/ubuntu/pedix/.env`, thay thế chữ `PLACEHOLDER` bằng các ID thật này! Sau đó chạy `sudo systemctl restart pedix-backend` để cập nhật.

#### Tạo Nhóm người dùng (Groups)
1. Bên trong User Pool, chuyển sang tab **Groups**.
2. Bấm **Create group**. Tên: `pedix-users`.
3. Tạo thêm một group nữa tên là `pedix-admins`.

---

### Bước 2: Tạo hàm Lambda Post-Confirmation 

Khi một người dùng đăng ký mới, chúng ta muốn tự động đẩy họ vào nhóm `pedix-users`.

1. Vào giao diện **AWS Lambda** và bấm **Create function**.
2. Chọn **Author from scratch**.
3. **Function name:** `Pedix-PostConfirmation`.
4. **Runtime:** Python 3.12.
5. **Execution role:** Chọn **Use an existing role** và chỉ định `Pedix-PostConfirmation-Role`.
6. Bấm **Create function**.
7. Ở khung soạn thảo Code source, dán đoạn mã Python này vào:

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
        print(f"Lỗi khi thêm user vào group: {e}")
        raise e
        
    return event
```
8. Bấm nút **Deploy**.
9. **Gắn vào Cognito:** Quay lại Cognito User Pool > tab **User pool properties** > **Lambda triggers** > **Add Lambda trigger**. Chọn loại **Post confirmation** và chỉ định hàm `Pedix-PostConfirmation` của bạn.

---

### Bước 3: Cơ sở dữ liệu DynamoDB

Pedix dùng chế độ **On-Demand** cho DynamoDB, nghĩa là nếu không có ai truy cập thì bạn không tốn 1 xu nào ($0).

Thực tế, bạn không cần phải tự tay tạo bảng! Khi backend FastAPI khởi động trên EC2, vòng đời ứng dụng (lifespan) trong file `main.py` sẽ sử dụng thư viện `boto3` để quét và tự động tạo các bảng nếu chưa có.

Để kiểm chứng:
1. Vào dịch vụ **DynamoDB**.
2. Bấm vào **Tables** ở thanh menu trái.
3. Bạn sẽ thấy 4 bảng đã được tự động tạo sẵn:
   * `pedix_sessions` (Lưu trữ tin nhắn chat)
   * `pedix_profiles` (Hồ sơ sức khỏe trẻ em)
   * `pedix_documents` (Quản lý file cơ sở tri thức)
   * `pedix_analytics_log` (Nhật ký phân tích cho Admin)

---

### Bước 4: Bảo vệ API Gateway bằng Cognito Authorizer

Bây giờ chúng ta sẽ ép API Gateway kiểm tra Token JWT của Cognito trước khi cho phép dữ liệu đi vào EC2.

1. Quay lại `Pedix-API` trong dịch vụ **API Gateway**.
2. Ở menu trái, chọn **Authorizers** > **Create New Authorizer**.
3. **Name:** `PedixCognitoAuthorizer`.
4. **Type:** Cognito.
5. **Cognito User Pool:** Chọn `pedix-user-pool`.
6. **Token Source:** Gõ chữ `Authorization`.
7. Bấm **Create**.
8. Trở lại phần **Resources**. Bấm vào phương thức `ANY` nằm dưới `/{proxy+}`.
9. Bấm vào **Method Request**.
10. Ở ô **Authorization**, bấm nút sổ xuống và chọn `PedixCognitoAuthorizer` (Bạn có thể cần F5 trang web nếu không thấy).
11. Bấm biểu tượng dấu tích để lưu lại.
12. **Quan trọng:** Bấm nút **Deploy API** lần nữa vào stage `prod` để áp dụng lớp bảo vệ này!
