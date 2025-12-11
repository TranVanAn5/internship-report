---
title : "Set up IDP Pipeline"
date :  "`r Sys.Date()`" 
weight : 8
chapter : false
pre : " <b> 5.8 </b> "
---

# Set up IDP Pipeline

In this section, you will setup SQS Worker to process documents through IDP (Intelligent Document Processing) pipeline.

---

## IDP Flow

```
Upload → S3 → DynamoDB (UPLOADED) → SQS
                                     ↓
                              EC2 Worker
                                     ↓
                    PyPDF2 (digital) / Textract (scanned)
                                     ↓
                         Chunk Text (1000 tokens)
                                     ↓
                    Cohere Embed Multilingual v3 (Bedrock)
                                     ↓
                         Qdrant (store vectors)
                                     ↓
                      DynamoDB (EMBEDDING_DONE)
```

> 💡 **Note**: Worker uses **PyPDF2** for digital PDF (text-based) and **Textract** for scanned PDF (image-based).

---

## Document States

| Status | Description |
|--------|-------------|
| `UPLOADED` | File uploaded, waiting for processing |
| `IDP_RUNNING` | Worker is processing |
| `TEXTRACT_DONE` | OCR completed (scanned PDF only) |
| `EMBEDDING_DONE` | Completed, ready to use |
| `FAILED` | Error occurred |

---

## Step 1: Access EC2 via Session Manager

```bash
# Get Instance ID
INSTANCE_ID=$(terraform -chdir=terraform output -raw ec2_instance_id)

# Connect
aws ssm start-session --target $INSTANCE_ID --region ap-southeast-1
```

After connecting:
```bash
sudo su - ec2-user
cd /home/ec2-user/backend
```

> 💡 **Note**: EC2 has 2 folders:
> - `app/` - Boilerplate from user_data script
> - `backend/` - Actual code deployed via CI/CD (contains run_worker.py)

---

## Step 2: Check Worker Code

Worker code is in `backend/run_worker.py`. Verify the file exists:

```bash
ls -la
# Must have: run_worker.py, app/, requirements.txt
```

![Worker Files](/images/5-Workshop/8-idp-pipeline/worker-files.png)

---

## Step 3: Configure Environment

Ensure `.env` file has all variables (in `backend/` folder):

```bash
cd /home/ec2-user/backend
cat .env
```

Important variables for IDP:
```
SQS_QUEUE_URL=https://sqs.ap-southeast-1.amazonaws.com/<account>/arc-dev-document-queue
S3_BUCKET=arc-documents-<account>
QDRANT_HOST=localhost
QDRANT_PORT=6333
AWS_REGION=ap-southeast-1
```

---

## Step 4: Start Worker

### Option A: Run directly (for debugging)

```bash
# Activate virtual environment (if available)
source venv/bin/activate

# Run worker
python run_worker.py
```

Worker will display:
```
============================================================
IDP Pipeline - SQS Worker
============================================================
Queue URL: https://sqs.ap-southeast-1.amazonaws.com/xxx/arc-dev-document-queue
Bucket: arc-documents-xxx
Region: ap-southeast-1
Qdrant: localhost:6333
------------------------------------------------------------
Processing indefinitely (Ctrl+C to stop)...
```

### Option B: Run in background with nohup

```bash
nohup python run_worker.py > worker.log 2>&1 &

# Check process
ps aux | grep run_worker

# View logs
tail -f worker.log
```

### Option C: Run in Docker (recommended)

```bash
# Add worker to docker-compose.yml
docker-compose up -d worker
```

---

## Step 5: Test IDP Pipeline

### 5.1 Upload test file to S3

```bash
# From local machine
aws s3 cp test-sample.pdf s3://arc-documents-<account>/uploads/test-001.pdf
```

### 5.2 Create record in DynamoDB

```bash
aws dynamodb put-item \
  --table-name arc-dev-documents \
  --item '{
    "doc_id": {"S": "test-001"},
    "sk": {"S": "METADATA"},
    "filename": {"S": "test-sample.pdf"},
    "s3_key": {"S": "uploads/test-001.pdf"},
    "status": {"S": "UPLOADED"},
    "uploaded_at": {"S": "'$(date -u +%Y-%m-%dT%H:%M:%SZ)'"}
  }'
```

### 5.3 Send message to SQS

```bash
aws sqs send-message \
  --queue-url https://sqs.ap-southeast-1.amazonaws.com/<account>/arc-dev-document-queue \
  --message-body '{
    "doc_id": "test-001",
    "s3_key": "uploads/test-001.pdf",
    "filename": "test-sample.pdf"
  }'
```

![Test Upload](/images/5-Workshop/8-idp-pipeline/test-upload.png)

---

## Step 6: Monitor Processing

View worker logs:

```bash
# If running directly
# Logs displayed on terminal

# If running in background
tail -f worker.log
```

Successful logs will look like:
```
2024-01-15 10:30:00 - INFO - Received message for doc_id: test-001
2024-01-15 10:30:01 - INFO - Downloading from S3: uploads/test-001.pdf
2024-01-15 10:30:02 - INFO - Extracting text with PyPDF2...
2024-01-15 10:30:03 - INFO - Created 8 chunks from document
2024-01-15 10:30:05 - INFO - Generating embeddings with Cohere...
2024-01-15 10:30:10 - INFO - Stored 8 vectors for test-001
2024-01-15 10:30:10 - INFO - Updated status: EMBEDDING_DONE
2024-01-15 10:30:10 - INFO - Document test-001 processed successfully
```

![Monitor Logs](/images/5-Workshop/8-idp-pipeline/monitor-logs.png)

---

## Step 7: Verify Processing

### 7.1 Check DynamoDB

```bash
aws dynamodb get-item \
  --table-name arc-dev-documents \
  --key '{"doc_id":{"S":"test-001"},"sk":{"S":"METADATA"}}' \
  --query 'Item.{status:status.S,chunks:chunk_count.N}'
```

Expected output:
```json
{
  "status": "EMBEDDING_DONE",
  "chunks": "8"
}
```

### 7.2 Check Qdrant

```bash
# On EC2
curl -s http://localhost:6333/collections/arc_documents/points/count | jq
```

Expected output:
```json
{
  "result": {
    "count": 8
  }
}
```

![Verify Processing](/images/5-Workshop/8-idp-pipeline/verify-processing.png)

---

## Error Handling

| Issue | Cause | Solution |
|-------|-------|----------|
| Worker not receiving messages | SQS URL incorrect | Check `.env` |
| Bedrock timeout | Rate limit | Increase retry delay |
| Qdrant connection refused | Container not started | `docker-compose up -d qdrant` |
| FAILED status | Check error_message in DynamoDB | Fix and retry |

### Retry Failed Document

```bash
# Update status back to UPLOADED to retry
aws dynamodb update-item \
  --table-name arc-dev-documents \
  --key '{"doc_id":{"S":"test-001"},"sk":{"S":"METADATA"}}' \
  --update-expression "SET #s = :s" \
  --expression-attribute-names '{"#s":"status"}' \
  --expression-attribute-values '{":s":{"S":"UPLOADED"}}'

# Send message to SQS again
aws sqs send-message \
  --queue-url $SQS_QUEUE_URL \
  --message-body '{"doc_id":"test-001","s3_key":"uploads/test-001.pdf","filename":"test-sample.pdf"}'
```

---

## Checklist

- [ ] Access EC2 via Session Manager
- [ ] Worker code is available
- [ ] Environment variables configured
- [ ] Worker is running
- [ ] Test document uploaded to S3
- [ ] SQS message sent
- [ ] Worker processed document (logs)
- [ ] Status = EMBEDDING_DONE in DynamoDB
- [ ] Vectors stored in Qdrant
