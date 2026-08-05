---
title: "3. Cài đặt EC2 & Backend"
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

# Thiết lập EC2, Qdrant & FastAPI Backend

Bước này sẽ hướng dẫn bạn khởi chạy máy chủ, cấu hình bảo mật mạng, và triển khai phần lõi FastAPI backend cùng với cơ sở dữ liệu vector Qdrant.

---

### Bước 1: Khởi chạy máy chủ EC2

1. Truy cập vào bảng điều khiển **EC2 Dashboard** và bấm **Launch instance**.
2. **Name:** Nhập `Pedix-Backend-Server`.
3. **Application and OS Images:** Chọn **Ubuntu** (Phiên bản *Ubuntu Server 26.04 LTS* hoặc tương tự).
4. **Instance type:** Chọn **t3.micro**.
5. **Key pair:** Bấm **Create new key pair**, đặt tên `pedix-ec2-key`, chọn **RSA** và **.pem**, sau đó bấm **Create key pair**. File `.pem` sẽ được tải về máy tính của bạn.
6. **Network settings:**
   * Chọn **Default VPC** và chọn bất kỳ Subnet nào (ví dụ: `ap-southeast-1a`).
   * **Auto-assign public IP:** Đảm bảo đang bật **Enable**.
   * Chọn **Create security group**. Đặt tên là `pedix-ec2-sg`.
   * **Inbound Security Group Rules:** 
     * Rule 1: Type `SSH`, Source type `Anywhere` (`0.0.0.0/0`).
7. **Configure storage:** Đổi thành **30 GiB** (gp3).
8. **Advanced details:**
   * **IAM instance profile:** Chọn `Pedix-EC2-Role` mà bạn đã tạo ở bước trước.
9. Bấm nút màu cam **Launch instance**.

---

### Bước 2: SSH và Cấu hình 2GB Swap Memory (Bộ nhớ ảo)

Vì máy chủ `t3.micro` chỉ có 1GB RAM, chúng ta phải tạo thêm bộ nhớ ảo (Swap) để hệ thống không bị sập (Out of memory) khi cài đặt các package Python nặng hoặc khi chạy Docker.

1. Mở PowerShell hoặc Terminal trên máy tính. Trỏ tới thư mục chứa file `pedix-ec2-key.pem` vừa tải về.
2. Sửa lại quyền file (Dành cho Windows PowerShell):
```powershell
icacls.exe "pedix-ec2-key.pem" /reset
icacls.exe "pedix-ec2-key.pem" /grant:r "$($env:USERNAME):(R)"
icacls.exe "pedix-ec2-key.pem" /inheritance:r
```
3. Kết nối SSH (thay `IP_ADDRESS` bằng địa chỉ Public IPv4 của EC2):
```bash
ssh -i "pedix-ec2-key.pem" ubuntu@IP_ADDRESS
```
4. Khi đã vào được EC2, chạy các lệnh sau để tạo 2GB Swap:
```bash
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

---

### Bước 3: Cài đặt Docker & Chạy cơ sở dữ liệu Qdrant

Chúng ta sẽ ép (bind) Qdrant chỉ được phép chạy trên `127.0.0.1` (localhost) để nó bị cô lập hoàn toàn khỏi mạng internet.

1. Cập nhật hệ điều hành và cài đặt Docker:
```bash
sudo apt update && sudo apt install -y git python3-venv python3-pip docker.io
sudo usermod -aG docker ubuntu
newgrp docker
```
2. Tạo thư mục lưu trữ dữ liệu vĩnh viễn cho Qdrant:
```bash
sudo mkdir -p /opt/pedicompass/qdrant_data
sudo chown -R $USER:$USER /opt/pedicompass
```
3. Chạy container Qdrant (Phiên bản 1.10.1):
```bash
docker run -d \
  --name pedix_qdrant \
  --restart unless-stopped \
  -p 127.0.0.1:6333:6333 \
  -v /opt/pedicompass/qdrant_data:/qdrant/storage \
  qdrant/qdrant:v1.10.1
```

---

### Bước 4: Tải Source code & Cài đặt thư viện

1. Clone mã nguồn từ GitHub về thư mục root:
```bash
cd /home/ubuntu
git clone https://github.com/Robertise/Pedix.git pedix
cd /home/ubuntu/pedix
```
2. Cài đặt môi trường ảo Python và tải thư viện (sử dụng cờ `--no-cache-dir` để tránh lỗi đầy dung lượng ổ cứng tạm thời):
```bash
python3 -m venv .venv
source .venv/bin/activate

pip install --no-cache-dir torch --index-url https://download.pytorch.org/whl/cpu
pip install --no-cache-dir -r backend/requirements.txt
pip install --no-cache-dir -r ingestion/requirements.txt
pip install email-validator
```

---

### Bước 5: Cấu hình biến môi trường `.env` và Khởi chạy Service

1. Chạy lệnh sau để tạo file biến môi trường (Lưu ý: Các giá trị Cognito sẽ được điền ở các bước sau):
```bash
cat << 'EOF' > /home/ubuntu/pedix/.env
AWS_REGION=ap-southeast-1
BEDROCK_MODEL_ID=global.anthropic.claude-haiku-4-5-20251001-v1:0
BEDROCK_HAIKU_MODEL_ID=global.anthropic.claude-haiku-4-5-20251001-v1:0
COGNITO_USER_POOL_ID=PLACEHOLDER
COGNITO_CLIENT_ID=PLACEHOLDER
COGNITO_REGION=ap-southeast-1
DYNAMODB_TABLE_PREFIX=pedix_
QDRANT_HOST=localhost
QDRANT_PORT=6333
QDRANT_COLLECTION=pedix_kb
FRONTEND_URL=*
EOF

cp /home/ubuntu/pedix/.env /home/ubuntu/pedix/backend/.env
```

2. Để đảm bảo server tự chạy lại sau khi khởi động máy, chúng ta tạo một `systemd` service. Gõ `sudo nano /etc/systemd/system/pedix-backend.service` và dán đoạn mã sau vào:
```ini
[Unit]
Description=Pedix FastAPI Backend Service
After=network.target

[Service]
User=ubuntu
Group=ubuntu
WorkingDirectory=/home/ubuntu/pedix/backend
Environment="PATH=/home/ubuntu/pedix/.venv/bin"
ExecStart=/home/ubuntu/pedix/.venv/bin/uvicorn main:app --host 0.0.0.0 --port 8000 --workers 1
Restart=always

[Install]
WantedBy=multi-user.target
```
> [!NOTE]
> Chúng ta bắt buộc phải dùng `--workers 1` vì tính năng SSE streaming lưu state bộ nhớ tạm. Nếu mở nhiều worker, request sẽ bị đá sang các worker khác nhau dẫn đến lỗi không tìm thấy luồng dữ liệu.

3. Kích hoạt và bật service:
```bash
sudo systemctl daemon-reload
sudo systemctl enable pedix-backend
sudo systemctl start pedix-backend
```

4. Kiểm tra xem service đã chạy thành công chưa:
```bash
curl http://127.0.0.1:8000/api/health
```
*(Bạn sẽ thấy kết quả trả về `{"status":"ok","service":"pedix-backend"}`)*.

Tuyệt vời! Backend của bạn đang chạy ở cổng 8000 của máy chủ. Tiếp theo chúng ta sẽ mở đường cho nó ra ngoài Internet thông qua API Gateway.