---
title: "6. Phân phối Frontend"
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

# Đẩy Frontend lên S3, CloudFront & Kiểm thử Hệ thống

Ở bước cuối cùng này, chúng ta sẽ biên dịch (build) mã nguồn React và đưa nó lên một bucket Amazon S3 bị khóa kín (private). Sau đó, dùng mạng phân phối nội dung toàn cầu Amazon CloudFront CDN để truyền tải trang web đến người dùng.

---

### Bước 1: Build mã nguồn React

1. Trên máy tính của bạn (không phải trên EC2), mở thư mục `frontend` của dự án Pedix.
2. Tạo một file tên là `.env.production` ở thư mục gốc của `frontend`.
3. Dán các biến sau vào, nhớ thay các ID bằng ID Cognito thực tế và đường dẫn API Gateway của bạn:

```env
VITE_API_BASE_URL=https://[ID_API_CỦA_BẠN].execute-api.ap-southeast-1.amazonaws.com/prod
VITE_COGNITO_USER_POOL_ID=ap-southeast-1_[POOL_CỦA_BẠN]
VITE_COGNITO_CLIENT_ID=[CLIENT_ID_CỦA_BẠN]
VITE_COGNITO_REGION=ap-southeast-1
```

> [!WARNING]
> **Cảnh báo lỗi mã hóa UTF-8 (Windows PowerShell):** Nếu bạn tạo file này bằng lệnh `echo` trên PowerShell, nó sẽ lưu dưới dạng UTF-16LE. Trình biên dịch Vite (Node.js) sẽ ngó lơ file này, khiến nút Đăng nhập/Đăng ký của bạn bấm vào không có phản ứng gì khi lên mạng. Hãy đảm bảo file được lưu với chuẩn **UTF-8**.

4. Chạy lệnh build:
```bash
npm install
npm run build
```
Hệ thống sẽ tạo ra một thư mục `dist` chứa các file tĩnh đã được tối ưu hóa cho môi trường Production.

---

### Bước 2: Tạo Amazon S3 Bucket (Private)

1. Vào dịch vụ **Amazon S3** và bấm **Create bucket**.
2. **Bucket name:** `pedix-frontend-prod-[tên-bạn]` (Tên bucket phải không trùng lặp trên toàn thế giới).
3. **Region:** `ap-southeast-1` (Singapore).
4. **Block Public Access settings:** Đảm bảo **Block all public access** đã được **TÍCH CHỌN** (Chúng ta muốn bucket này bị khóa hoàn toàn với internet).
5. Bấm **Create bucket**.
6. Mở bucket vừa tạo, bấm **Upload**, kéo thả toàn bộ các file nằm *bên trong* thư mục `dist` vào, rồi bấm **Upload**.

---

### Bước 3: Tạo Amazon CloudFront Distribution

1. Vào dịch vụ **CloudFront** và bấm **Create Distribution**.
2. **Origin domain:** Bấm mũi tên xổ xuống và chọn S3 bucket của bạn.
3. **Origin access:** Chọn **Origin access control settings (recommended)**. 
   * Bấm **Create control setting** và giữ nguyên mặc định.
   * **QUAN TRỌNG:** CloudFront sẽ báo rằng bạn cần cập nhật Policy cho S3 bucket. Bấm **Copy policy**. Lát nữa chúng ta sẽ dán nó.
4. **Default cache behavior:**
   * Viewer protocol policy: Đổi thành **Redirect HTTP to HTTPS**.
5. **Web Application Firewall (WAF):** Chọn **Do not enable security protections** (để tránh phát sinh thêm chi phí).
6. Bấm **Create distribution**.

#### Cập nhật S3 Bucket Policy (OAC)
1. Quay lại S3 bucket của bạn > Chọn tab **Permissions**.
2. Cuộn xuống phần **Bucket policy** và bấm **Edit**.
3. Dán đoạn policy bạn vừa copy bên CloudFront vào. Đoạn code này chỉ cấp quyền đọc duy nhất cho CloudFront, còn lại chặn đứng tất cả. Bấm **Save changes**.

---

### Bước 4: Cấu hình trang lỗi SPA Fallback

Vì React là một Single Page Application (SPA), mọi định tuyến (router) đều do trình duyệt tự xử lý. Nếu người dùng gõ thẳng đường dẫn `/chat` lên thanh địa chỉ, CloudFront sẽ chui vào S3 tìm file tên là `chat` và ném ra lỗi 403 Forbidden.

1. Vào CloudFront Distribution của bạn, chuyển sang tab **Error pages**.
2. Bấm **Create custom error response**.
3. HTTP error code: `403: Forbidden`.
4. Customize error response: **Yes**.
5. Response page path: `/index.html`.
6. HTTP Response code: `200: OK`.
7. Bấm **Create custom error response**.
8. Lặp lại chính xác các bước trên nhưng áp dụng cho mã lỗi `404: Not Found`.

---

### Bước 5: Hoàn tất & Nghiệm thu Hệ thống

Hệ thống của bạn đã hoàn thiện 100%!

Hãy đợi vài phút để CloudFront hoàn tất triển khai (Deploying). Copy đường dẫn **Distribution domain name** (VD: `https://d2bx3usq72976a.cloudfront.net`) và dán vào trình duyệt.

**Checklist Nghiệm thu End-to-End:**
- [x] Trang web load tốc độ cao, có ổ khóa bảo mật HTTPS.
- [x] Tạo thành công tài khoản và nhận được email chứa mã xác nhận.
- [x] Đăng nhập trơn tru (Chứng tỏ `.env.production` cấu hình đúng).
- [x] Khung chat RAG phản hồi chữ realtime qua SSE Streaming chạy dọc xuyên suốt mạng VPC nội bộ.

> [!TIP]
> **Xóa Cache (Invalidation):** Sau này nếu bạn sửa code và upload lại file lên S3, bạn bắt buộc phải vào CloudFront > **Invalidations** > **Create invalidation** > nhập `/*` để bắt các máy chủ vệ tinh toàn cầu xóa cache cũ và lấy mã nguồn mới.

**Chúc mừng bạn đã tự tay triển khai thành công một hệ thống RAG Y khoa siêu bảo mật trên AWS!**