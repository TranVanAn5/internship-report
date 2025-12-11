---
title : "Data Preparation"
date :  "`r Sys.Date()`" 
weight : 5
chapter : false
pre : " <b> 5.5 </b> "
---

# Data Preparation

Before creating AWS resources, you need to download sample data to test the system.

## Step 1: Download Sample Data

1. Access [ARC Sample Data](https://drive.google.com/drive/folders/1JdnSBW-dkbO6PxRSv8DLYvnBBSfiTgAQ?usp=sharing)
2. Download the data to your computer
3. Extract the file, which will create a folder named `DATA`

![Download Data](/images/5-Workshop/5-data-preparation/data-prepare.png)

### Document Requirements

| Limit | Value |
|-------|-------|
| Format | PDF (text-based or scanned) |
| Max size | 50 MB |
| Max pages | 500 pages |
| Recommended | 10-100 pages |

---

# Prepare AWS Resources

## Step 2: Create S3 Bucket

S3 Bucket is used to store uploaded PDF documents.

1. **Search for S3 in AWS Console**

![S3 Search](/images/5-Workshop/5-data-preparation/1.1.jpg)

2. **Click Create bucket**

![S3 Console](/images/5-Workshop/5-data-preparation/3.png)

3. **Configure bucket:**
   - **Bucket name**: `arc-documents-<YOUR-ACCOUNT-ID>` (replace `<YOUR-ACCOUNT-ID>` with your AWS Account ID)
   - **AWS Region**: `Asia Pacific (Singapore) ap-southeast-1`
   - Keep other settings as default

![S3 Create Bucket](/images/5-Workshop/5-data-preparation/bucket-create.png)
![S3 Create Bucket](/images/5-Workshop/5-data-preparation/bucket-create-1.png)
![S3 Create Bucket](/images/5-Workshop/5-data-preparation/bucket-create-2.png)

4. **Click Create bucket**

> 💡 **Tip**: To get your AWS Account ID, run:
> ```bash
> aws sts get-caller-identity --query Account --output text
> ```

**Or create using CLI:**

```bash
aws s3 mb s3://arc-documents-$(aws sts get-caller-identity --query Account --output text) --region ap-southeast-1
```

## Step 3: Create DynamoDB Table

DynamoDB Table is used to store document metadata.

1. **Search for DynamoDB in AWS Console**

![DynamoDB Search](/images/5-Workshop/5-data-preparation/dynamodb-search.png)

2. **Click Create table**

![DynamoDB Console](/images/5-Workshop/5-data-preparation/dynamodb-console.png)

3. **Configure table:**
   - **Table name**: `arc-documents`
   - **Partition key**: `doc_id` (String)
   - **Sort key**: `sk` (String)
   - **Table settings**: Default settings

![DynamoDB Create Table](/images/5-Workshop/5-data-preparation/dynamodb-create-table.png)

4. **Click Create table**

**Or create using CLI:**

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

## Step 4: Create SQS Queue

SQS Queue is used for IDP pipeline to process documents.

1. **Search for SQS in AWS Console**

![SQS Search](/images/5-Workshop/5-data-preparation/sqs-search.png)

2. **Click Create queue**

![SQS Console](/images/5-Workshop/5-data-preparation/sqs-console.png)

3. **Configure queue:**
   - **Type**: Standard
   - **Name**: `arc-document-queue`
   - Keep other settings as default

![SQS Create Queue](/images/5-Workshop/5-data-preparation/sqs-create-queue.png)

4. **Click Create queue**

**Or create using CLI:**

```bash
aws sqs create-queue --queue-name arc-document-queue --region ap-southeast-1
```

## Step 5: Verify Resources

Check that all resources have been created:

```bash
# S3 Bucket
aws s3 ls | grep arc-documents

# DynamoDB Table
aws dynamodb describe-table --table-name arc-documents --region ap-southeast-1 --query "Table.TableName"

# SQS Queue
aws sqs get-queue-url --queue-name arc-document-queue --region ap-southeast-1
```

## Step 6: Upload Data to S3

Upload PDF files from the `DATA` folder downloaded in Step 1:

```bash
# Upload all files from DATA folder
aws s3 cp DATA/ s3://arc-documents-<YOUR-ACCOUNT-ID>/uploads/ --recursive

# Or upload individual file
aws s3 cp DATA/sample-document.pdf s3://arc-documents-<YOUR-ACCOUNT-ID>/uploads/
```

> 💡 **Tip**: Replace `<YOUR-ACCOUNT-ID>` with your AWS Account ID

Verify upload:

```bash
aws s3 ls s3://arc-documents-<YOUR-ACCOUNT-ID>/uploads/
```

## Checklist

Before proceeding, make sure:

- [ ] AWS CLI installed and configured
- [ ] Terraform installed
- [ ] Docker installed and running
- [ ] Node.js 18+ installed
- [ ] Python 3.11+ installed
- [ ] Git installed
- [ ] Repository cloned
- [ ] IAM user created with sufficient permissions
- [ ] Bedrock models approved (Claude + Cohere)
- [ ] S3 Bucket created
- [ ] DynamoDB Table created
- [ ] SQS Queue created
- [ ] Sample documents uploaded to S3
