---
title : "Preparation"
date :  "`r Sys.Date()`" 
weight : 2
chapter : false
pre : " <b> 5.2 </b> "
---

## Prerequisites

Requirements needed to complete this workshop:

- **AWS Client Machine**: Configured with access to necessary AWS services
- **Development Environment**: Windows, macOS, or Linux with basic development tools
- **Basic Knowledge**: Understanding of AWS, Python, JavaScript, and Docker
- **GitHub Account**: To clone source code and track changes
- **AWS Budget**: Approximately $65/month for resources (EC2, Bedrock, NAT Gateway)

## Tool Installation

### 1. AWS CLI

AWS Command Line Interface (AWS CLI) is a tool to interact with AWS services.

**Windows:**
```powershell
# Download and install MSI installer
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

Terraform is an Infrastructure as Code tool to provision AWS resources.

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

Docker to run Qdrant vector database locally and on EC2.

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

Node.js for frontend development with React + Vite.

**Using nvm (recommended):**
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

Python for backend FastAPI application.

**Windows:** Download from [python.org](https://www.python.org/downloads/) (select "Add to PATH")

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

Git for version control.

**Windows:** Download from [git-scm.com](https://git-scm.com/download/win)

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

## Configure AWS Credentials

### Create IAM User

1. Login to [AWS Console](https://console.aws.amazon.com/)
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

Enter information:
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

## Activate Amazon Bedrock Models

### Request Model Access

1. AWS Console → **Amazon Bedrock** → **Model access**
2. Click **Manage model access**
3. Select models:
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

## Prepare Sample Documents

Project comes with sample PDFs in `samples/`:

```bash
ls samples/
# data-structures-sample.pdf
# test-sample.pdf
```

### Document Requirements

| Limit | Value |
|-------|-------|
| Format | PDF (text-based or scanned) |
| Max size | 50 MB |
| Max pages | 500 pages |
| Recommended | 10-100 pages |

## Checklist

Before proceeding, ensure:

- [ ] AWS CLI installed and configured
- [ ] Terraform installed
- [ ] Docker installed and running
- [ ] Node.js 18+ installed
- [ ] Python 3.11+ installed
- [ ] Git installed
- [ ] Repository cloned
- [ ] IAM user created with proper permissions
- [ ] Bedrock models approved (Claude + Cohere)
- [ ] Sample documents ready