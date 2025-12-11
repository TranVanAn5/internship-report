---
title : "Dọn dẹp Tài nguyên"
date :  "`r Sys.Date()`" 
weight : 12
chapter : false
pre : " <b> 5.12 </b> "
---

# Dọn dẹp Tài nguyên

Sau khi hoàn thành workshop, dọn dẹp AWS resources để tránh phát sinh chi phí.

> ⚠️ **Cảnh báo**: Các bước này sẽ XÓA VĨNH VIỄN tất cả data và resources!

---

## Thứ tự Dọn dẹp

1. Stop services trên EC2
2. Empty S3 buckets
3. Terraform destroy
4. Verify cleanup

---

## Bước 1: Stop Services trên EC2

Kết nối EC2 qua Session Manager:

```bash
INSTANCE_ID=$(terraform -chdir=terraform output -raw ec2_instance_id)
aws ssm start-session --target $INSTANCE_ID --region ap-southeast-1
```

Stop Docker containers:

```bash
sudo su - ec2-user
cd /home/ec2-user/app

# Stop containers
docker-compose down

# Remove volumes
docker volume rm app_qdrant_storage
```
---

## Bước 2: Empty S3 Buckets

S3 buckets phải empty trước khi Terraform destroy:

```bash
# Lấy bucket name từ Terraform output
BUCKET=$(terraform -chdir=terraform output -raw s3_bucket_name)

# Empty bucket
aws s3 rm s3://$BUCKET --recursive

# Hoặc force delete
aws s3 rb s3://$BUCKET --force
```

---

## Bước 3: Terraform Destroy

```bash
cd terraform
terraform plan -destroy
terraform destroy
```

Nhập `yes` khi được hỏi. Quá trình này mất khoảng 10-15 phút.


---

## Bước 4: Manual Cleanup (nếu cần)

Nếu còn resources chưa bị xóa:

```bash
# CloudWatch Log Groups
aws logs describe-log-groups --log-group-name-prefix /aws/arc | jq -r '.logGroups[].logGroupName' | xargs -I {} aws logs delete-log-group --log-group-name {}

# EC2 Key Pair (nếu tạo manual)
aws ec2 delete-key-pair --key-name arc-keypair

# Amplify App (nếu tạo manual)
aws amplify list-apps | jq -r '.apps[] | select(.name | contains("arc")) | .appId' | xargs -I {} aws amplify delete-app --app-id {}
```

---

## Bước 5: Verify Cleanup

```bash
# Check EC2
aws ec2 describe-instances --filters "Name=tag:Name,Values=*arc*" --query 'Reservations[].Instances[].InstanceId'

# Check S3
aws s3 ls | grep arc

# Check DynamoDB
aws dynamodb list-tables --query 'TableNames[?contains(@, `arc`)]'

# Check Cognito
aws cognito-idp list-user-pools --max-results 20 --query 'UserPools[?contains(Name, `arc`)]'
```

Tất cả commands trên không nên trả về kết quả.

---

## Bước 6: Check Costs

1. Mở [AWS Billing Dashboard](https://console.aws.amazon.com/billing/)
2. Kiểm tra **Bills** cho tháng hiện tại
3. Set up **Budgets** alert cho tương lai

---

## Xử lý Lỗi

| Lỗi | Giải pháp |
|-----|-----------|
| DependencyViolation | Destroy theo thứ tự: `terraform destroy -target=module.amplify` trước |
| BucketNotEmpty | `aws s3 rb s3://bucket-name --force` |
| DeleteConflict (IAM) | Detach policies trước khi delete role |
| Resource in use | Đợi vài phút rồi retry |

---

## Kết luận

Chúc mừng bạn đã hoàn thành workshop **Academic Research Chatbot (ARC)**! 🎉

### Những gì bạn đã học:

- Triển khai RAG chatbot trên AWS
- Sử dụng Amazon Bedrock (Claude 3.5 Sonnet + Cohere Embed)
- Xây dựng IDP pipeline với PyPDF2/Textract
- Vector search với Qdrant
- Authentication với Cognito
- Infrastructure as Code với Terraform

### Tài nguyên bổ sung:

- [AWS Bedrock Documentation](https://docs.aws.amazon.com/bedrock/)
- [Qdrant Documentation](https://qdrant.tech/documentation/)
- [FastAPI Documentation](https://fastapi.tiangolo.com/)

---

## Checklist

- [ ] Stop Docker containers trên EC2
- [ ] Empty S3 buckets
- [ ] Terraform destroy thành công
- [ ] Verify không còn resources
- [ ] Check billing

---

**Cảm ơn bạn đã tham gia workshop!** 🙏
