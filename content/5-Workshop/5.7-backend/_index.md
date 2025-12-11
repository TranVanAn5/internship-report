---
title : "Set up Backend API"
date :  "`r Sys.Date()`" 
weight : 7
chapter : false
pre : " <b> 5.7 </b> "
---

# Set up Backend API

In this section, you will configure the Backend API (FastAPI) and Qdrant vector database on EC2.

## Backend Architecture

```
Internet → ALB (:80) → EC2 Private Subnet
                        ├── FastAPI Container (:8000)
                        ├── Qdrant Container (:6333)
                        └── SQS Worker (background)
```

> 💡 **Note**: EC2 is located in a Private Subnet with no Public IP. Access via **SSM Session Manager**.

---

## Step 1: Access EC2 via Session Manager

The EC2 instance was created in a Private Subnet with no Public IP. Use AWS Systems Manager Session Manager to access it.

### Method 1: AWS Console

1. Open **AWS Console** → **EC2** → **Instances**
2. Select instance `arc-dev-app-server`
3. Click **Connect** → **Session Manager** → **Connect**

![SSM Connect](/images/5-Workshop/7-backend/ssm-connect.png)

### Method 2: AWS CLI

```bash
# Get Instance ID from Terraform output
INSTANCE_ID=$(terraform -chdir=terraform output -raw ec2_instance_id)

# Connect via SSM
aws ssm start-session --target $INSTANCE_ID --region ap-southeast-1
```

> ⚠️ **Requirement**: Install [Session Manager Plugin](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager-working-with-install-plugin.html)

---

## Step 2: Check Running Services

EC2 has been automatically set up via **user_data script** when Terraform created the instance. Verify the services:

```bash
# Switch to ec2-user
sudo su - ec2-user

# Check Docker containers
docker ps
```

You should see 2 containers running:
- `app-fastapi-1` - FastAPI server (port 8000)
- `app-qdrant-1` - Qdrant vector database (port 6333)

![Docker PS](/images/5-Workshop/7-backend/docker-ps.png)

```bash
# Check Qdrant
curl http://localhost:6333/collections

# Check FastAPI
curl http://localhost:8000/health
```

---

## Step 3: Deploy Backend Code

Backend code will be deployed via **CI/CD Pipeline** (CodePipeline → CodeBuild → CodeDeploy). However, for quick testing, you can deploy manually:

```bash
cd /home/ec2-user

# Clone repository
git clone https://github.com/CrystalJohn/ARC-project.git
cd ARC-project/backend

# Stop old containers
cd /home/ec2-user/app
docker-compose down

# Copy backend code
cp -r /home/ec2-user/ARC-project/backend/* /home/ec2-user/app/

# Start with new code
docker-compose up -d --build
```

---

## Step 4: Configure Environment Variables

Create a `.env` file with values from Terraform outputs:

```bash
cd /home/ec2-user/app

# Get values from Terraform outputs (run on local machine)
# terraform -chdir=terraform output

cat > .env << 'EOF'
# AWS Configuration
AWS_REGION=ap-southeast-1

# S3
S3_BUCKET_NAME=arc-documents-<account-id>

# DynamoDB
DYNAMODB_TABLE_NAME=arc-dev-documents

# SQS
SQS_QUEUE_URL=https://sqs.ap-southeast-1.amazonaws.com/<account-id>/arc-dev-document-queue

# Qdrant (local container)
QDRANT_HOST=qdrant
QDRANT_PORT=6333

# Cognito
COGNITO_USER_POOL_ID=ap-southeast-1_xxxxx
COGNITO_CLIENT_ID=xxxxx

# Bedrock
BEDROCK_MODEL_ID=anthropic.claude-3-5-sonnet-20241022-v2:0
EMBEDDING_MODEL_ID=amazon.titan-embed-text-v2:0
EOF
```

> 💡 **Tip**: Replace `<account-id>` and `xxxxx` values with actual outputs from Terraform.

---

## Step 5: Restart Services

```bash
# Restart to load new .env
docker-compose down
docker-compose up -d

# Check logs
docker-compose logs -f fastapi
```
---

## Step 6: Verify via ALB

Backend is exposed via Application Load Balancer. Verify from your local machine:

```bash
# Get ALB DNS from Terraform output
ALB_DNS=$(terraform -chdir=terraform output -raw alb_dns_name)

# Test health endpoint
curl http://$ALB_DNS/health
# {"status":"healthy"}
```

## Step 7: Check Qdrant Collection

```bash
# On EC2
curl http://localhost:6333/collections

# Create collection for documents (if not exists)
curl -X PUT http://localhost:6333/collections/documents \
  -H "Content-Type: application/json" \
  -d '{
    "vectors": {
      "size": 1024,
      "distance": "Cosine"
    }
  }'
```

> 💡 **Note**: Vector size 1024 corresponds to Amazon Titan Embeddings v2.

---

## Checklist

- [ ] Successfully accessed EC2 via Session Manager
- [ ] Docker containers running (fastapi, qdrant)
- [ ] `.env` file configured
- [ ] Health check via ALB successful
- [ ] Qdrant collection created

---

## Troubleshooting

### Unable to connect to Session Manager

```bash
# Check SSM Agent on EC2
sudo systemctl status amazon-ssm-agent

# Verify IAM Role has AmazonSSMManagedInstanceCore policy
```

### Container fails to start

```bash
# View logs
docker-compose logs

# Check disk space
df -h
```

### ALB health check fails

```bash
# Verify Security Group allows port 8000 from ALB
# Verify FastAPI is listening on 0.0.0.0:8000
docker-compose logs fastapi
```
