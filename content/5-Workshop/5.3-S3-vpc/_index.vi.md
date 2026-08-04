---
title: "Bước 1: EC2, Qdrant & Backend FastAPI"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

Trong bước này, chúng ta khởi tạo và cấu hình hạ tầng điện toán chính bao gồm máy chủ **FastAPI** và cơ sở dữ liệu **Qdrant Vector DB** chạy trên Docker tại **Amazon EC2**.

---

### 1. Khởi tạo EC2 Instance
1. Truy cập EC2 Console và tạo máy chủ mới:
   - **Tên:** `pedix-backend-server`
   - **AMI:** Ubuntu Server 24.04 LTS (64-bit x86)
   - **Instance Type:** `t3.micro` (2 vCPU, 1 GiB RAM)
   - **IAM Role:** `Pedix-EC2-Role`
   - **Dung lượng:** 30 GiB ổ cứng EBS gp3

---

### 2. Cấu hình EBS Swap (Chống lỗi OOM)
Kết nối SSH vào EC2 và chạy lệnh:
```bash
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

---

### 3. Triển khai Qdrant Vector DB (Docker)
Cài đặt Docker và khởi chạy container Qdrant:
```bash
sudo apt-get update && sudo apt-get install -y docker.io
sudo systemctl enable --now docker
sudo docker run -d --name qdrant -p 6333:6333 -v qdrant_storage:/qdrant/storage qdrant/qdrant:v1.7.4
```

---

### 4. Chạy Máy chủ FastAPI
Tải nguồn code, cài đặt gói thư viện (bản PyTorch CPU-only) và kích hoạt Uvicorn:
```bash
git clone https://github.com/Robertise/Pedix.code.git /home/ubuntu/app
cd /home/ubuntu/app
pip install -r requirements.txt --extra-index-url https://download.pytorch.org/whl/cpu
uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
```