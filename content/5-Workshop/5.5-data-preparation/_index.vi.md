---
title : "Chuẩn bị Dữ liệu"
date :  "`r Sys.Date()`" 
weight : 5
chapter : false
pre : " <b> 5.5 </b> "
---

# Chuẩn bị Dữ liệu

Trước khi tạo AWS resources, bạn cần tải xuống tập dữ liệu mẫu để test hệ thống.

## Bước 1: Tải Xuống Tập Dữ liệu

1. Truy cập [ARC Sample Data](https://drive.google.com/drive/folders/1JdnSBW-dkbO6PxRSv8DLYvnBBSfiTgAQ?usp=sharing)
2. Tải dữ liệu về máy tính của bạn
3. Giải nén file, sẽ tạo ra một thư mục có tên `DATA`

![Download Data](/images/5-Workshop/5-data-preparation/data-prepare.png)

### Yêu cầu Documents

| Limit | Value |
|-------|-------|
| Format | PDF (text-based hoặc scanned) |
| Max size | 50 MB |
| Max pages | 500 pages |
| Recommended | 10-100 pages |

---

# Chuẩn bị AWS Resources

## Bước 2: Tạo S3 Bucket

S3 Bucket dùng để lưu trữ documents PDF được upload.

1. **Tìm kiếm S3 trong AWS Console**

![S3 Search](/images/5-Workshop/5-data-preparation/1.1.jpg)

2. **Click Create bucket**

![S3 Console](/images/5-Workshop/5-data-preparation/3.png)

3. **Cấu hình bucket:**
   - **Bucket name**: `arc-documents-<YOUR-ACCOUNT-ID>` (thay `<YOUR-ACCOUNT-ID>` bằng AWS Account ID của bạn)
   - **AWS Region**: `Asia Pacific (Singapore) ap-southeast-1`
   - Giữ các settings khác mặc định

![S3 Create Bucket](/images/5-Workshop/5-data-preparation/s3-create-bucket.png)

4. **Click Create bucket**

> 💡 **Tip**: Để lấy AWS Account ID, chạy:
> ```bash
> aws sts get-caller-identity --query Account --output text
> ```

**Hoặc tạo bằng CLI:**

```bash
aws s3 mb s3://arc-documents-$(aws sts get-caller-identity --query Account --output text) --region ap-southeast-1
```

## Bước 3: Tạo DynamoDB Table

DynamoDB Table dùng để lưu metadata của documents.

1. **Tìm kiếm DynamoDB trong AWS Console**

![DynamoDB Search](/images/5-Workshop/5-data-preparation/dynamodb-search.png)

2. **Click Create table**

![DynamoDB Console](/images/5-Workshop/5-data-preparation/dynamodb-console.png)

3. **Cấu hình table:**
   - **Table name**: `arc-documents`
   - **Partition key**: `doc_id` (String)
   - **Sort key**: `sk` (String)
   - **Table settings**: Default settings

![DynamoDB Create Table](/images/5-Workshop/5-data-preparation/dynamodb-create-table.png)

4. **Click Create table**

**Hoặc tạo bằng CLI:**

```bash
aws dynamodb create-table \
  --table-name arc-documents \
  --attribute-definitions \
    AttributeName=doc_id,AttributeType=S \
    AttributeName=sk,AttributeType=S \
  --key-schema \
    AttributeName=doc_id,KeyType=HASH \
    AttributeName=sk,KeyType=RANGE \
  --billing-mode PAY_PER_REQUEST \
  --region ap-southeast-1
```

## Bước 4: Tạo SQS Queue

SQS Queue dùng cho IDP pipeline xử lý documents.

1. **Tìm kiếm SQS trong AWS Console**

![SQS Search](/images/5-Workshop/5-data-preparation/sqs-search.png)

2. **Click Create queue**

![SQS Console](/images/5-Workshop/5-data-preparation/sqs-console.png)

3. **Cấu hình queue:**
   - **Type**: Standard
   - **Name**: `arc-document-queue`
   - Giữ các settings khác mặc định

![SQS Create Queue](/images/5-Workshop/5-data-preparation/sqs-create-queue.png)

4. **Click Create queue**

**Hoặc tạo bằng CLI:**

```bash
aws sqs create-queue --queue-name arc-document-queue --region ap-southeast-1
```

## Bước 5: Verify Resources

Kiểm tra tất cả resources đã được tạo:

```bash
# S3 Bucket
aws s3 ls | grep arc-documents

# DynamoDB Table
aws dynamodb describe-table --table-name arc-documents --region ap-southeast-1 --query "Table.TableName"

# SQS Queue
aws sqs get-queue-url --queue-name arc-document-queue --region ap-southeast-1
```

![Verify Resources](/images/5-Workshop/5-data-preparation/verify-resources.png)

## Bước 6: Upload Dữ liệu lên S3

Upload các file PDF từ thư mục `DATA` đã tải về ở Bước 1:

```bash
# Upload tất cả files từ thư mục DATA
aws s3 cp DATA/ s3://arc-documents-<YOUR-ACCOUNT-ID>/uploads/ --recursive

# Hoặc upload từng file
aws s3 cp DATA/sample-document.pdf s3://arc-documents-<YOUR-ACCOUNT-ID>/uploads/
```

> 💡 **Tip**: Thay `<YOUR-ACCOUNT-ID>` bằng AWS Account ID của bạn

Verify upload:

```bash
aws s3 ls s3://arc-documents-<YOUR-ACCOUNT-ID>/uploads/
```

![S3 Upload](/images/5-Workshop/5-data-preparation/s3-upload-verify.png)

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
- [ ] S3 Bucket created
- [ ] DynamoDB Table created
- [ ] SQS Queue created
- [ ] Sample documents uploaded to S3

3. Cấu hình bucket:
   - **Bucket name**: `arc-documents-<YOUR-ACCOUNT-ID>` (thay `<YOUR-ACCOUNT-ID>` bằng AWS Account ID của bạn)
   - **AWS Region**: `Asia Pacific (Singapore) ap-southeast-1`
   - Giữ các settings khác mặc định

![S3 Create Bucket](images/s3-create-bucket.png)

4. Click **Create bucket**

> 💡 **Tip**: Để lấy AWS Account ID, chạy: `aws sts get-caller-identity --query Account --output text`

**Hoặc tạo bằng CLI:**
```bash
aws s3 mb s3://arc-documents-$(aws sts get-caller-identity --query Account --output text) --region ap-southeast-1
```

### Bước 3: Tạo DynamoDB Table

DynamoDB Table dùng để lưu metadata của documents.

1. Tìm kiếm **DynamoDB** trong AWS Console

![DynamoDB Search](images/dynamodb-search.png)

2. Click **Create table**

![DynamoDB Console](images/dynamodb-console.png)

3. Cấu hình table:
   - **Table name**: `arc-documents`
   - **Partition key**: `doc_id` (String)
   - **Sort key**: `sk` (String)
   - **Table settings**: Default settings

![DynamoDB Create Table](images/dynamodb-create-table.png)

4. Click **Create table**

**Hoặc tạo bằng CLI:**
```bash
aws dynamodb create-table \
  --table-name arc-documents \
  --attribute-definitions \
    AttributeName=doc_id,AttributeType=S \
    AttributeName=sk,AttributeType=S \
  --key-schema \
    AttributeName=doc_id,KeyType=HASH \
    AttributeName=sk,KeyType=RANGE \
  --billing-mode PAY_PER_REQUEST \
  --region ap-southeast-1
```

### Bước 4: Tạo SQS Queue

SQS Queue dùng cho IDP pipeline xử lý documents.

1. Tìm kiếm **SQS** trong AWS Console

![SQS Search](images/sqs-search.png)

2. Click **Create queue**

![SQS Console](images/sqs-console.png)

3. Cấu hình queue:
   - **Type**: Standard
   - **Name**: `arc-document-queue`
   - Giữ các settings khác mặc định

![SQS Create Queue](images/sqs-create-queue.png)

4. Click **Create queue**

**Hoặc tạo bằng CLI:**
```bash
aws sqs create-queue --queue-name arc-document-queue --region ap-southeast-1
```

### Bước 5: Verify Resources

Kiểm tra tất cả resources đã được tạo:

```bash
# S3 Bucket
aws s3 ls | grep arc-documents

# DynamoDB Table
aws dynamodb describe-table --table-name arc-documents --region ap-southeast-1 --query "Table.TableName"

# SQS Queue
aws sqs get-queue-url --queue-name arc-document-queue --region ap-southeast-1
```

![Verify Resources](images/verify-resources.png)

### Bước 6: Upload Dữ liệu lên S3

Upload các file PDF từ thư mục `DATA` đã tải về ở Bước 1:

```bash
# Upload tất cả files từ thư mục DATA
aws s3 cp DATA/ s3://arc-documents-<YOUR-ACCOUNT-ID>/uploads/ --recursive

# Hoặc upload từng file
aws s3 cp DATA/sample-document.pdf s3://arc-documents-<YOUR-ACCOUNT-ID>/uploads/
```

> 💡 **Tip**: Thay `<YOUR-ACCOUNT-ID>` bằng AWS Account ID của bạn

Verify upload:

```bash
aws s3 ls s3://arc-documents-<YOUR-ACCOUNT-ID>/uploads/
```

![S3 Upload](images/s3-upload-verify.png)

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
