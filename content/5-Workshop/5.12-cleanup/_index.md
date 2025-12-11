---
title : "Cleanup Resources"
date :  "`r Sys.Date()`" 
weight : 12
chapter : false
pre : " <b> 5.12 </b> "
---

# Cleanup Resources

After completing the workshop, clean up AWS resources to avoid incurring charges.

> ⚠️ **Warning**: These steps will PERMANENTLY DELETE all data and resources!

---

## Cleanup Order

1. Stop services on EC2
2. Empty S3 buckets
3. Terraform destroy
4. Verify cleanup

---

## Step 1: Stop Services on EC2

Connect EC2 via Session Manager:

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

## Step 2: Empty S3 Buckets

S3 buckets must be empty before Terraform destroy:

```bash
# Get bucket name from Terraform output
BUCKET=$(terraform -chdir=terraform output -raw s3_bucket_name)

# Empty bucket
aws s3 rm s3://$BUCKET --recursive

# Or force delete
aws s3 rb s3://$BUCKET --force
```

---

## Step 3: Terraform Destroy

```bash
cd terraform
terraform plan -destroy
terraform destroy
```

Enter `yes` when prompted. This process takes about 10-15 minutes.

---

## Step 4: Manual Cleanup (if needed)

If there are still resources not deleted:

```bash
# CloudWatch Log Groups
aws logs describe-log-groups --log-group-name-prefix /aws/arc | jq -r '.logGroups[].logGroupName' | xargs -I {} aws logs delete-log-group --log-group-name {}

# EC2 Key Pair (if created manually)
aws ec2 delete-key-pair --key-name arc-keypair

# Amplify App (if created manually)
aws amplify list-apps | jq -r '.apps[] | select(.name | contains("arc")) | .appId' | xargs -I {} aws amplify delete-app --app-id {}
```

---

## Step 5: Verify Cleanup

```bash
# Check EC2
aws ec2 describe-instances --filters "Name=tag:Name,Values=*arc*" --query 'Reservations[].Instances[].InstanceId'

# Check S3
aws s3 ls | grep arc

# Check RDS/DynamoDB
aws dynamodb list-tables --query 'TableNames[?contains(@, `arc`)]'

# Check Lambda
aws lambda list-functions --query 'Functions[?contains(FunctionName, `arc`)].FunctionName'

# Check ECR
aws ecr describe-repositories --query 'repositories[?contains(repositoryName, `arc`)].repositoryName'
```

Expected output: All empty.

---

## Cost Estimation

Before cleanup, check estimated costs:

```bash
# CloudWatch - Check billing dashboard for actual charges
# https://console.aws.amazon.com/billing/
```

---

## Checklist

- [ ] EC2 services stopped
- [ ] S3 buckets emptied
- [ ] Terraform destroy completed
- [ ] Manual cleanup verified
- [ ] All resources deleted
- [ ] Billing dashboard checked
