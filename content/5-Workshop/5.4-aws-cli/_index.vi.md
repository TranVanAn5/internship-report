---
title : "Cấu hình AWS CLI"
date :  "`r Sys.Date()`" 
weight : 4
chapter : false
pre : " <b> 5.4 </b> "
---

# Cấu hình AWS CLI

Để triển khai và quản lý giải pháp, bạn cần cấu hình AWS Command Line Interface (AWS CLI) với thông tin xác thực của mình.

## Các bước

### Bước 1: Kiểm tra AWS CLI đã cài đặt

```bash
aws --version
```

Nếu chưa có, cài đặt:

```powershell
# Download và cài đặt MSI installer
msiexec.exe /i https://awscli.amazonaws.com/AWSCLIV2.msi
```

### Bước 2: Tạo IAM User (nếu chưa có)

1. Đăng nhập AWS Console
2. Tìm kiếm "IAM" → Click IAM
3. Sidebar trái → Users → Create user
4. User name: `arc-workshop-user`
5. Click Next
6. Attach policies directly, chọn các policies:
   - AmazonEC2FullAccess
   - AmazonS3FullAccess
   - AmazonDynamoDBFullAccess
   - AmazonCognitoPowerUser
   - AmazonSQSFullAccess
   - AmazonTextractFullAccess
   - AmazonBedrockFullAccess
   - CloudWatchFullAccess
   - IAMFullAccess
7. Click Create user

### Bước 3: Tạo Access Key

1. Vào user vừa tạo → Tab Security credentials
2. Scroll xuống Access keys → Click Create access key
3. Chọn Command Line Interface (CLI)
4. Tick "I understand..." → Next
5. Description: `ARC Workshop CLI`
6. Click Create access key

**⚠️ QUAN TRỌNG:** Copy hoặc download .csv file

```
Access key ID: AKIAXXXXXXXXXXXXXXXX
Secret access key: xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

### Bước 4: Configure AWS CLI

Mở PowerShell và chạy:

```bash
aws configure
```

Nhập thông tin:

```
AWS Access Key ID [None]: AKIAXXXXXXXXXXXXXXXX
AWS Secret Access Key [None]: xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
Default region name [None]: ap-southeast-1
Default output format [None]: json
```

### Bước 5: Verify Configuration

Kiểm tra identity:

```bash
aws sts get-caller-identity
```

Output mong đợi:

```json
{
    "UserId": "AIDAXXXXXXXXXXXXXXXXX",
    "Account": "123456789012",
    "Arn": "arn:aws:iam::123456789012:user/arc-workshop-user"
}
```

