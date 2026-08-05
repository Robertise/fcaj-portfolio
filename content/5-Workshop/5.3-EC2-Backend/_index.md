---
title: "3. EC2 & Backend Setup"
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

# EC2, Qdrant & FastAPI Backend Setup

This step covers launching our compute engine, configuring network security, and deploying the core FastAPI backend alongside the Qdrant vector database.

---

### Step 1: Launch the EC2 Instance

1. Navigate to the **EC2 Dashboard** and click **Launch instance**.
2. **Name:** Enter `Pedix-Backend-Server`.
3. **Application and OS Images:** Select **Ubuntu** (Choose *Ubuntu Server 26.04 LTS* or similar).
4. **Instance type:** Select **t3.micro**.
5. **Key pair:** Click **Create new key pair**, name it `pedix-ec2-key`, select **RSA** and **.pem**, then click **Create key pair**. The `.pem` file will download to your computer.
6. **Network settings:**
   * Select your **Default VPC** and choose any Subnet (e.g., `ap-southeast-1a`).
   * **Auto-assign public IP:** Ensure this is **Enable**.
   * Select **Create security group**. Name it `pedix-ec2-sg`.
   * **Inbound Security Group Rules:** 
     * Rule 1: Type `SSH`, Source type `Anywhere` (`0.0.0.0/0`).
7. **Configure storage:** Change to **30 GiB** (gp3).
8. **Advanced details:**
   * **IAM instance profile:** Select the `Pedix-EC2-Role` you created in the prerequisites.
9. Click **Launch instance**.

---

### Step 2: SSH and 2GB Swap Memory Configuration

Since the `t3.micro` instance only has 1GB of RAM, we must create virtual Swap memory to prevent the system from crashing out of memory (OOM) when compiling Python packages or running Docker.

1. Open PowerShell or Terminal on your local machine. Navigate to the folder where `pedix-ec2-key.pem` was downloaded.
2. Fix permissions (Windows PowerShell):
```powershell
icacls.exe "pedix-ec2-key.pem" /reset
icacls.exe "pedix-ec2-key.pem" /grant:r "$($env:USERNAME):(R)"
icacls.exe "pedix-ec2-key.pem" /inheritance:r
```
3. Connect via SSH (replace `IP_ADDRESS` with your EC2 Public IPv4 address):
```bash
ssh -i "pedix-ec2-key.pem" ubuntu@IP_ADDRESS
```
4. Once inside the EC2, allocate 2GB of Swap memory:
```bash
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

---

### Step 3: Install Docker & Run Qdrant Database

We will bind Qdrant strictly to `127.0.0.1` so it is isolated from the network.

1. Update packages and install Docker:
```bash
sudo apt update && sudo apt install -y git python3-venv python3-pip docker.io
sudo usermod -aG docker ubuntu
newgrp docker
```
2. Create persistent storage for Qdrant:
```bash
sudo mkdir -p /opt/pedicompass/qdrant_data
sudo chown -R $USER:$USER /opt/pedicompass
```
3. Run the Qdrant container (Version 1.10.1):
```bash
docker run -d \
  --name pedix_qdrant \
  --restart unless-stopped \
  -p 127.0.0.1:6333:6333 \
  -v /opt/pedicompass/qdrant_data:/qdrant/storage \
  qdrant/qdrant:v1.10.1
```

---

### Step 4: Clone Code & Install Dependencies

1. Clone the project repository into the root home folder:
```bash
cd /home/ubuntu
git clone https://github.com/Robertise/Pedix.git pedix
cd /home/ubuntu/pedix
```
2. Set up the Python Virtual Environment and install packages (using `--no-cache-dir` to avoid disk quota errors):
```bash
python3 -m venv .venv
source .venv/bin/activate

pip install --no-cache-dir torch --index-url https://download.pytorch.org/whl/cpu
pip install --no-cache-dir -r backend/requirements.txt
pip install --no-cache-dir -r ingestion/requirements.txt
pip install email-validator
```

---

### Step 5: Configure `.env` and Start the Service

1. Run this command to create the environment file. (Note: We will fill in the Cognito values later):
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

2. To ensure the backend stays online even if the server reboots, create a `systemd` service. Type `sudo nano /etc/systemd/system/pedix-backend.service` and paste:
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
> We use `--workers 1` because our SSE streaming endpoint uses an in-memory `PendingRequestStore`. Using multiple workers causes request ID mismatch errors.

3. Enable and start the service:
```bash
sudo systemctl daemon-reload
sudo systemctl enable pedix-backend
sudo systemctl start pedix-backend
```

4. Verify it is running:
```bash
curl http://127.0.0.1:8000/api/health
```
*(You should see `{"status":"ok","service":"pedix-backend"}`)*.

Your backend is now running locally on port 8000. Next, we will expose it securely using an Internal Load Balancer and API Gateway!