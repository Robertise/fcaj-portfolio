---
title: "2. Chuẩn bị & Phân quyền IAM"
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

# Chuẩn bị & Thiết lập Phân quyền IAM Roles

Trước khi khởi tạo các hạ tầng cốt lõi, chúng ta cần thiết lập đúng AWS Region (khu vực), cấp quyền truy cập vào các mô hình AI của Amazon Bedrock, và tạo các Role phân quyền (IAM Roles) để đảm bảo các dịch vụ có thể giao tiếp bảo mật với nhau mà không cần dùng Access Key tĩnh (hardcode).

---

### Bước 1: Lựa chọn AWS Region

Chúng ta sẽ triển khai toàn bộ tài nguyên tại Region **Singapore**.
1. Đăng nhập vào [AWS Management Console](https://console.aws.amazon.com/).
2. Ở góc trên cùng bên phải, bấm vào menu thả xuống chọn Region.
3. Chọn **Asia Pacific (Singapore) `ap-southeast-1`**.

> [!WARNING]
> Bắt buộc phải giữ nguyên region `ap-southeast-1` trong suốt bài workshop này. Tài nguyên AWS (như VPC, máy chủ EC2) không thể nhìn thấy xuyên khu vực (cross-region).

---

### Bước 2: Kích hoạt mô hình AI Amazon Bedrock

Hệ thống Pedix sử dụng mô hình Claude 3 Haiku của Anthropic để suy luận y tế và băm nhỏ (chunking) tài liệu.

1. Trên thanh tìm kiếm của AWS Console, gõ **Bedrock** và chọn dịch vụ này.
2. Ở thanh menu bên trái, cuộn xuống và bấm vào **Model access** (nằm dưới mục Bedrock settings).
3. Bấm vào nút màu cam **Enable specific models** (hoặc **Manage model access**).
4. Tích chọn ô vuông bên cạnh **Anthropic** > **Claude 3 Haiku**.
5. Cuộn xuống dưới cùng và bấm **Save changes**. 
6. Đợi vài phút. Cột trạng thái sẽ chuyển thành chữ màu xanh **Access granted**.

---

### Bước 3: Tạo `Pedix-EC2-Role` (Dành cho Backend Server)

Để tuân thủ nguyên tắc bảo mật của AWS Well-Architected Framework, chúng ta tuyệt đối *không bao giờ* dán AWS Access Key vào file `.env` trên EC2. Thay vào đó, chúng ta sẽ gán một IAM Role cho máy chủ EC2.

1. Tìm kiếm dịch vụ **IAM** trên thanh tìm kiếm và mở nó.
2. Bấm vào **Roles** ở menu bên trái, sau đó chọn **Create role**.
3. Tại phần **Trusted entity type**, chọn **AWS service**.
4. Tại phần **Use case**, chọn **EC2**, rồi bấm **Next**.
5. Ở ô tìm kiếm quyền, gõ và tích chọn hộp kiểm bên cạnh:
   * `AmazonSSMManagedInstanceCore` *(Quyền này cho phép chúng ta SSH an toàn vào server thông qua Session Manager mà không cần mở cổng 22 ra internet).*
6. Bấm **Next**, đặt tên Role là **`Pedix-EC2-Role`**, và bấm **Create role**.

#### Gán Custom Inline Policy (Chính sách mở rộng)
Máy chủ EC2 cần quyền truy cập Bedrock và thao tác với DynamoDB.
1. Bấm vào tên Role vừa tạo **`Pedix-EC2-Role`**.
2. Ở tab **Permissions**, bấm **Add permissions** > **Create inline policy**.
3. Chuyển sang thẻ **JSON** và dán đoạn mã sau:

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
4. Bấm **Next**, đặt tên chính sách là **`Pedix-EC2-Permissions`**, rồi bấm **Create policy**.

---

### Bước 4: Tạo `Pedix-PostConfirmation-Role` (Dành cho AWS Lambda)

Role này sẽ được cấp cho một hàm AWS Lambda để nó có thể tự động xếp người dùng mới đăng ký vào trong một group của Cognito.

1. Trở lại trang IAM > **Roles** > bấm **Create role**.
2. Tại **Trusted entity type**, chọn **AWS service**.
3. Tại **Use case**, chọn **Lambda**, rồi bấm **Next**.
4. Tìm và tích chọn **`AWSLambdaBasicExecutionRole`** *(cho phép Lambda đẩy log lỗi lên CloudWatch)*.
5. Bấm **Next**, đặt tên là **`Pedix-PostConfirmation-Role`**, và bấm **Create role**.

#### Gán Custom Inline Policy
1. Bấm vào Role **`Pedix-PostConfirmation-Role`**.
2. Bấm **Add permissions** > **Create inline policy**.
3. Chuyển sang thẻ **JSON** và dán:

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
4. Bấm **Next**, đặt tên là **`Pedix-Cognito-GroupAssignment`**, và bấm **Create policy**.

Vậy là phần chuẩn bị đã xong. Bạn đã sẵn sàng để triển khai hệ thống đám mây thực tế!