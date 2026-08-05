---
title: "Blog 1: API Gateway & VPC Link"
weight: 10
---

# Bảo Mật Backend: Tích Hợp API Gateway, VPC Link v2 và Internal ALB

**Xem bài viết đã đăng trên AWS Study Group:** [Chèn link tại đây](https://www.facebook.com/groups/awsstudygroupfcj/...)

*Bài chia sẻ AWS Study Group – First Cloud AI Journey (FCAJ)*  
*Dựa trên: [Build scalable REST APIs using Amazon API Gateway private integration with Application Load Balancer](https://aws.amazon.com/blogs/compute/build-scalable-rest-apis-using-amazon-api-gateway-private-integration-with-application-load-balancer) – AWS Compute Blog*

---

## 1. Bài toán bảo mật Backend

Khi triển khai một dịch vụ backend trên AWS - như hệ thống FastAPI + Qdrant mà mình xây dựng - các tiêu chuẩn bảo mật khắt khe yêu cầu máy chủ EC2 phải nằm trong private subnet. Mục tiêu đặt ra: **Backend không được mở bất kỳ port public nào cho ứng dụng**. Mọi request từ bên ngoài bắt buộc phải đi qua một chuỗi kiểm soát và không được truy cập trực tiếp.

## 2. Vì sao chọn VPC Link v2 + ALB trực tiếp thay vì qua NLB?

Một kiến trúc phổ biến trước đây là: `API Gateway -> VPC Link -> NLB -> ALB -> Backend`. Cách làm này tốn thêm một network hop và một Load Balancer (NLB) chỉ để làm cầu nối. Hơn nữa, do NLB hoạt động ở Layer 4, nó gặp hạn chế trong việc định tuyến theo đường dẫn (path-based routing).

Bài viết trên AWS Compute Blog giới thiệu một giải pháp tối ưu hơn: **API Gateway REST API có thể tích hợp thẳng vào Internal ALB thông qua VPC Link v2**. Cách này loại bỏ hoàn toàn NLB trung gian, trong khi vẫn giữ được sức mạnh định tuyến Layer 7 của ALB (như path rule, health check HTTP). Một VPC Link v2 còn có thể trỏ tới nhiều ALB/NLB cùng lúc.

![Blog 1 Illustration](/images/blogs/blog1.png)

## 3. Kiến trúc bảo mật đã triển khai

```text
CloudFront -> API Gateway (kèm Cognito Authorizer)
           -> VPC Link v2 
           -> Internal ALB 
           -> Target Group :8000 (Healthy)
           -> EC2 private subnet (FastAPI + Qdrant localhost only)
```

**Điểm mấu chốt về bảo mật:** Security Group của máy chủ backend chỉ được phép mở inbound port 8000 từ **Security Group của ALB**, tuyệt đối không mở cho dải IP `0.0.0.0/0`. Dùng Security Group làm source giúp rule luôn đúng kể cả khi ALB thay đổi IP bên dưới.

## 4. Thực nghiệm: Lỗi Resource Shadowing với `{proxy+}`

Trong quá trình triển khai thực tế, route wildcard thường (`/api/{proxy+}`, Buffered) hoạt động rất tốt. Tuy nhiên, khi mình thêm một route tường minh riêng cho SSE streaming - `/api/chat/stream` (Response Transfer Mode = **Stream** để trả dữ liệu token-by-token) - thì mọi request khác nằm dưới `/api/chat/...` (ví dụ `/api/chat/message`) bỗng nhiên trả về lỗi 404.

**Nguyên nhân:** Việc tạo một resource tường minh (như `/api/chat/stream`) khiến API Gateway tự động sinh ra một resource cha (`/api/chat`). Resource cha này vô tình "che khuất" (shadow) route wildcard `{proxy+}` vốn đang xử lý các sub-path khác.

**Cách xử lý:** Mình phải tạo thêm một route `{proxy+}` con nằm ngay dưới nhánh cha `/api/chat/`, trỏ về cùng VPC Link (Buffered) và deploy lại API stage. Kết quả là cả route streaming và các route REST thông thường đều hoạt động song song.

**Bài học rút ra:** API Gateway không đơn giản là "path cứ thế đi thẳng". Một resource tường minh ở nhánh con có thể phá vỡ wildcard route ở cấp cha, đòi hỏi phải đồng bộ thủ công.

## 5. Các vấn đề vận hành khác

- **Lỗi mất state của Uvicorn:** Chạy `uvicorn --workers 2` gây lỗi mất state của Server-Sent Events (`Request ID expired`) do bộ nhớ lưu trữ request (in-memory) không được share giữa các process. Mình đã phải hạ về `--workers 1`.
- **Rớt kết nối SSE:** CloudFront và ALB rất dễ ngắt kết nối SSE nếu vòng lặp AI suy luận quá lâu. Mình giải quyết bằng cách giảm `heartbeat_interval` từ 15s xuống 5s để liên tục giữ kết nối "sống".

## 6. Kết luận

Sự khác biệt giữa "kết nối chạy được" và "kết nối chuẩn Zero-Trust" nằm ở chi tiết nhỏ: cấu hình đúng nguồn Security Group, chọn đúng Response Transfer Mode, và hiểu rõ cách API Gateway xử lý độ ưu tiên giữa wildcard và đường dẫn tường minh. Đối với backend cần giấu kín trong private subnet, pattern **API Gateway + VPC Link v2 + Internal ALB** là một lựa chọn cực kỳ đáng giá.

---

**Tham khảo:**
- Vijay Menon, Christian Silva – *Build scalable REST APIs using Amazon API Gateway private integration with Application Load Balancer*, AWS Compute Blog. [Link](https://aws.amazon.com/blogs/compute/build-scalable-rest-apis-using-amazon-api-gateway-private-integration-with-application-load-balancer)