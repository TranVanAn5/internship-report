---
title : "Configure AWS CLI"
date :  "`r Sys.Date()`" 
weight : 4
chapter : false
pre : " <b> 5.4 </b> "
---

To deploy and manage the solution, you need to configure the AWS Command Line Interface (AWS CLI) with your authentication information.

## Steps

### Step 1: Check if AWS CLI is installed

```bash
aws --version
```

If not installed:

```powershell
# Download and install MSI installer
msiexec.exe /i https://awscli.amazonaws.com/AWSCLIV2.msi
```

### Step 2: Create IAM User (if not already exists)

1. Login to AWS Console
2. Search "IAM" → Click IAM
3. Left sidebar → Users → Create user
4. User name: `arc-workshop-user`
5. Click Next
6. Attach policies directly, select policies:
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

### Step 3: Create Access Key

1. Go to the created user → Tab Security credentials
2. Scroll down Access keys → Click Create access key
3. Select Command Line Interface (CLI)
4. Tick "I understand..." → Next
5. Description: `ARC Workshop CLI`
6. Click Create access key

**⚠️ IMPORTANT:** Copy or download .csv file

```
Access key ID: AKIAXXXXXXXXXXXXXXXX
Secret access key: xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

### Step 4: Configure AWS CLI

Open PowerShell and run:

```bash
aws configure
```

Enter information:

```
AWS Access Key ID [None]: AKIAXXXXXXXXXXXXXXXX
AWS Secret Access Key [None]: xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
Default region name [None]: ap-southeast-1
Default output format [None]: json
```

### Step 5: Verify Configuration

Check identity:

```bash
aws sts get-caller-identity
```

Expected output:

```json
{
    "UserId": "AIDAXXXXXXXXXXXXXXXXX",
    "Account": "123456789012",
    "Arn": "arn:aws:iam::123456789012:user/arc-workshop-user"
}
```