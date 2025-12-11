---
title : "Thiết lập Backend API"
date :  "`r Sys.Date()`" 
weight : 7
chapter : false
pre : " <b> 5.7 </b> "
---

# Thiết lập Backend API

Trong phần này, bạn sẽ cấu hình Backend API (FastAPI) và Qdrant vector database trên EC2.

## Kiến trúc Backend

```
Internet → ALB (:80) → EC2 Private Subnet
                        ├── FastAPI Container (:8000)
                        ├── Qdrant Container (:6333)
                        └── SQS Worker (background)
```

> 💡 **Note**: EC2 nằm trong Private Subnet, không có Public IP. Truy cập qua **SSM Session Manager**.

---

## Bước 1: Truy cập EC2 qua Session Manager

EC2 instance đã được tạo trong Private Subnet và không có Public IP. Sử dụng AWS Systems Manager Session Manager để truy cập.

### Cách 1: AWS Console

1. Mở **AWS Console** → **EC2** → **Instances**
2. Chọn instance `arc-dev-app-server`
3. Click **Connect** → **Session Manager** → **Connect**

![SSM Connect](/images/5-Workshop/7-backend/ssm-connect.png)

### Cách 2: AWS CLI

```bash
# Lấy Instance ID từ Terraform output
INSTANCE_ID=$(terraform -chdir=terraform output -raw ec2_instance_id)

# Kết nối qua SSM
aws ssm start-session --target $INSTANCE_ID --region ap-southeast-1
```

> ⚠️ **Yêu cầu**: Cài đặt [Session Manager Plugin](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager-working-with-install-plugin.html)

---

## Bước 2: Kiểm tra Services đã chạy

EC2 đã được setup tự động qua **user_data script** khi Terraform tạo instance. Kiểm tra các services:

```bash
# Chuyển sang ec2-user
sudo su - ec2-user

# Kiểm tra Docker containers
docker ps
```

Bạn sẽ thấy 2 containers đang chạy:
- `app-fastapi-1` - FastAPI server (port 8000)
- `app-qdrant-1` - Qdrant vector database (port 6333)

![Docker PS](/images/5-Workshop/7-backend/docker-ps.png)

```bash
# Kiểm tra Qdrant
curl http://localhost:6333/collections

# Kiểm tra FastAPI
curl http://localhost:8000/health
```

---

## Bước 3: Deploy Backend Code

Backend code sẽ được deploy qua **CI/CD Pipeline** (CodePipeline → CodeBuild → CodeDeploy). Tuy nhiên, để test nhanh, bạn có thể deploy thủ công:

```bash
cd /home/ec2-user

# Clone repository
git clone https://github.com/CrystalJohn/ARC-project.git
cd ARC-project/backend

# Stop containers cũ
cd /home/ec2-user/app
docker-compose down

# Copy backend code
cp -r /home/ec2-user/ARC-project/backend/* /home/ec2-user/app/

# Start với code mới
docker-compose up -d --build
```

---

## Bước 4: Cấu hình Environment Variables

Tạo file `.env` với các giá trị từ Terraform outputs:

```bash
cd /home/ec2-user/app

# Lấy values từ Terraform outputs (chạy trên máy local)
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

> 💡 **Tip**: Thay `<account-id>` và các giá trị `xxxxx` bằng outputs thực tế từ Terraform.

---

## Bước 5: Restart Services

```bash
# Restart để load .env mới
docker-compose down
docker-compose up -d

# Kiểm tra logs
docker-compose logs -f fastapi
```
---

## Bước 6: Verify qua ALB

Backend được expose qua Application Load Balancer. Kiểm tra từ máy local:

```bash
# Lấy ALB DNS từ Terraform output
ALB_DNS=$(terraform -chdir=terraform output -raw alb_dns_name)

# Test health endpoint
curl http://$ALB_DNS/health
# {"status":"healthy"}
```

## Bước 7: Kiểm tra Qdrant Collection

```bash
# Trên EC2
curl http://localhost:6333/collections

# Tạo collection cho documents (nếu chưa có)
curl -X PUT http://localhost:6333/collections/documents \
  -H "Content-Type: application/json" \
  -d '{
    "vectors": {
      "size": 1024,
      "distance": "Cosine"
    }
  }'
```

> 💡 **Note**: Vector size 1024 tương ứng với Amazon Titan Embeddings v2.

---

## Checklist

- [ ] Truy cập EC2 qua Session Manager thành công
- [ ] Docker containers đang chạy (fastapi, qdrant)
- [ ] File `.env` đã được cấu hình
- [ ] Health check qua ALB thành công
- [ ] Qdrant collection đã được tạo

---

## Troubleshooting

### Không thể kết nối Session Manager

```bash
# Kiểm tra SSM Agent trên EC2
sudo systemctl status amazon-ssm-agent

# Kiểm tra IAM Role có policy AmazonSSMManagedInstanceCore
```

### Container không start

```bash
# Xem logs
docker-compose logs

# Kiểm tra disk space
df -h
```

### ALB health check fail

```bash
# Kiểm tra Security Group cho phép port 8000 từ ALB
# Kiểm tra FastAPI đang listen trên 0.0.0.0:8000
docker-compose logs fastapi
```
