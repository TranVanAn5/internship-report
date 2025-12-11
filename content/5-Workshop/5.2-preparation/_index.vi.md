---
title : "Các bước chuẩn bị"
date :  "`r Sys.Date()`" 
weight : 2
chapter : false
pre : " <b> 5.2 </b> "
---

## Yêu cầu Tiên quyết

Các yêu cầu cần có để thực hiện workshop này:

- **Máy tính khách AWS**: Được cấu hình với quyền truy cập vào các dịch vụ AWS cần thiết
- **Môi trường phát triển**: Windows, macOS hoặc Linux với các công cụ development cơ bản
- **Kiến thức cơ bản**: Hiểu biết về AWS, Python, JavaScript và Docker
- **Tài khoản GitHub**: Để clone source code và theo dõi changes
- **Ngân sách AWS**: Khoảng $65/tháng cho các resources (EC2, Bedrock, NAT Gateway)

## Cài đặt công cụ

### 1. AWS CLI

AWS Command Line Interface (AWS CLI) là công cụ để tương tác với AWS services.

**Windows:**
```powershell
# Download và cài đặt MSI installer
msiexec.exe /i https://awscli.amazonaws.com/AWSCLIV2.msi
```

**macOS:**
```bash
brew install awscli
```

**Linux:**
```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

Verify installation:
```bash
aws --version
# aws-cli/2.x.x Python/3.x.x
```

### 2. Terraform

Terraform là công cụ Infrastructure as Code để provision AWS resources.

**Windows:**
```powershell
choco install terraform
```

**macOS:**
```bash
brew tap hashicorp/tap
brew install hashicorp/tap/terraform
```

**Linux:**
```bash
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install terraform
```

Verify:
```bash
terraform --version
```

### 3. Docker

Docker để chạy Qdrant vector database locally và trên EC2.

**Windows/macOS:** Download [Docker Desktop](https://www.docker.com/products/docker-desktop/)

**Linux:**
```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER
```

Verify:
```bash
docker --version
docker run hello-world
```

### 4. Node.js (>= 18)

Node.js cho frontend development với React + Vite.

**Sử dụng nvm (khuyến nghị):**
```bash
# Install nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash

# Install Node.js 18
nvm install 18
nvm use 18
```

Verify:
```bash
node --version  # v18.x.x
npm --version   # 9.x.x
```

### 5. Python (>= 3.11)

Python cho backend FastAPI application.

**Windows:** Download từ [python.org](https://www.python.org/downloads/) (chọn "Add to PATH")

**macOS:**
```bash
brew install python@3.11
```

**Linux:**
```bash
sudo apt update
sudo apt install python3.11 python3.11-venv python3-pip
```

Verify:
```bash
python --version  # Python 3.11.x
pip --version
```

### 6. Git

Git cho version control.

**Windows:** Download từ [git-scm.com](https://git-scm.com/download/win)

**macOS:**
```bash
brew install git
```

**Linux:**
```bash
sudo apt install git
```

Verify:
```bash
git --version
```

## Clone Repository

```bash
git clone https://github.com/CrystalJohn/ARC-project.git
cd ARC-project
```

## Cấu hình AWS Credentials

### Tạo IAM User

1. Đăng nhập [AWS Console](https://console.aws.amazon.com/)
2. Navigate to **IAM** → **Users** → **Create user**
3. User name: `arc-workshop-user`
4. Attach policies:
   - AmazonEC2FullAccess
   - AmazonS3FullAccess
   - AmazonDynamoDBFullAccess
   - AmazonCognitoPowerUser
   - AmazonSQSFullAccess
   - AmazonTextractFullAccess
   - AmazonBedrockFullAccess
   - CloudWatchFullAccess
   - IAMFullAccess
5. Create access key → Download credentials

### Configure AWS CLI

```bash
aws configure
```

Nhập thông tin:
```
AWS Access Key ID: AKIAXXXXXXXXXXXXXXXX
AWS Secret Access Key: xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
Default region name: ap-southeast-1
Default output format: json
```

Verify:
```bash
aws sts get-caller-identity
```

## Kích hoạt Amazon Bedrock Models

### Request Model Access

1. AWS Console → **Amazon Bedrock** → **Model access**
2. Click **Manage model access**
3. Chọn models:
   - **Anthropic - Claude 3.5 Sonnet** (`anthropic.claude-3-5-sonnet-20241022-v2:0`)
   - **Cohere - Embed Multilingual v3** (`cohere.embed-multilingual-v3`)
4. Click **Request model access** → Accept Terms → **Submit**

### Verify Access

```bash
# Test Claude
aws bedrock get-foundation-model \
  --model-identifier anthropic.claude-3-5-sonnet-20241022-v2:0 \
  --region ap-southeast-1

# Test Cohere
aws bedrock get-foundation-model \
  --model-identifier cohere.embed-multilingual-v3 \
  --region ap-southeast-1
```

Expected: Status **Access granted**

## Chuẩn bị Sample Documents

Project có sẵn sample PDFs trong `samples/`:

```bash
ls samples/
# data-structures-sample.pdf
# test-sample.pdf
```

### Yêu cầu documents

| Limit | Value |
|-------|-------|
| Format | PDF (text-based hoặc scanned) |
| Max size | 50 MB |
| Max pages | 500 pages |
| Recommended | 10-100 pages |

## Checklist

Trước khi tiếp tục, đảm bảo:

- [ ] AWS CLI installed và configured
- [ ] Terraform installed
- [ ] Docker installed và running
- [ ] Node.js 18+ installed
- [ ] Python 3.11+ installed
- [ ] Git installed
- [ ] Repository cloned
- [ ] IAM user created với đủ permissions
- [ ] Bedrock models được approve (Claude + Cohere)
- [ ] Sample documents sẵn sàng

