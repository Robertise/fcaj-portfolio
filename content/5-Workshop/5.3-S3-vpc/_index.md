---
title: "Step 1: EC2, Qdrant & FastAPI Backend"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

In this step, we launch and configure the primary backend compute instance running **FastAPI** and the **Qdrant Vector Database** inside Docker on **Amazon EC2**.

---

### 1. Launch Amazon EC2 Instance
1. Open the EC2 Console and launch a new instance:
   - **Name:** `pedix-backend-server`
   - **AMI:** Ubuntu Server 24.04 LTS (64-bit x86)
   - **Instance Type:** `t3.micro` (2 vCPU, 1 GiB RAM)
   - **IAM Role:** `Pedix-EC2-Role`
   - **Storage:** 30 GiB gp3 EBS volume

---

### 2. Configure EBS Swap Space (Prevent OOM)
SSH into your EC2 instance and run:
```bash
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

---

### 3. Deploy Qdrant Vector DB (Docker)
Install Docker and start the Qdrant container:
```bash
sudo apt-get update && sudo apt-get install -y docker.io
sudo systemctl enable --now docker
sudo docker run -d --name qdrant -p 6333:6333 -v qdrant_storage:/qdrant/storage qdrant/qdrant:v1.7.4
```

---

### 4. Deploy FastAPI Server
Clone the repository, install Python dependencies (with PyTorch CPU-only build), and launch Uvicorn:
```bash
git clone https://github.com/Robertise/Pedix.code.git /home/ubuntu/app
cd /home/ubuntu/app
pip install -r requirements.txt --extra-index-url https://download.pytorch.org/whl/cpu
uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
```