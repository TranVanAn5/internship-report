---
title : "Thiết lập IDP Pipeline"
date :  "`r Sys.Date()`" 
weight : 8
chapter : false
pre : " <b> 5.8 </b> "
---

Trong phần này, bạn sẽ setup SQS Worker để xử lý documents qua IDP (Intelligent Document Processing) pipeline.

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

> 💡 **Note**: Worker sử dụng **PyPDF2** cho PDF digital (text-based) và **Textract** cho PDF scanned (image-based).

---

## Document States

| Status | Description |
|--------|-------------|
| `UPLOADED` | File đã upload, đang chờ xử lý |
| `IDP_RUNNING` | Worker đang xử lý |
| `TEXTRACT_DONE` | OCR hoàn tất (chỉ cho scanned PDF) |
| `EMBEDDING_DONE` | Hoàn tất, sẵn sàng sử dụng |
| `FAILED` | Có lỗi xảy ra |

---

## Bước 1: Truy cập EC2 qua Session Manager

```bash
# Lấy Instance ID
INSTANCE_ID=$(terraform -chdir=terraform output -raw ec2_instance_id)

# Kết nối
aws ssm start-session --target $INSTANCE_ID --region ap-southeast-1
```

Sau khi kết nối:
```bash
sudo su - ec2-user
cd /home/ec2-user/backend
```

> 💡 **Note**: Trên EC2 có 2 folders:
> - `app/` - Boilerplate từ user_data script
> - `backend/` - Code thực tế được deploy qua CI/CD (chứa run_worker.py)

---

## Bước 2: Kiểm tra Worker Code

Worker code nằm trong `backend/run_worker.py`. Kiểm tra file đã có:

```bash
ls -la
# Phải có: run_worker.py, app/, requirements.txt
```

![Worker Files](/images/5-Workshop/8-idp-pipeline/worker-files.png)

---

## Bước 3: Cấu hình Environment

Đảm bảo file `.env` có đầy đủ các biến (trong folder `backend/`):

```bash
cd /home/ec2-user/backend
cat .env
```

Các biến quan trọng cho IDP:
```
SQS_QUEUE_URL=https://sqs.ap-southeast-1.amazonaws.com/<account>/arc-dev-document-queue
S3_BUCKET=arc-documents-<account>
QDRANT_HOST=localhost
QDRANT_PORT=6333
AWS_REGION=ap-southeast-1
```

---

## Bước 4: Start Worker

### Option A: Chạy trực tiếp (để debug)

```bash
# Activate virtual environment (nếu có)
source venv/bin/activate

# Chạy worker
python run_worker.py
```

Worker sẽ hiển thị:
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

### Option B: Chạy trong background với nohup

```bash
nohup python run_worker.py > worker.log 2>&1 &

# Kiểm tra process
ps aux | grep run_worker

# Xem logs
tail -f worker.log
```

### Option C: Chạy trong Docker (recommended)

```bash
# Thêm worker vào docker-compose.yml
docker-compose up -d worker
```

---

## Bước 5: Test IDP Pipeline

### 5.1 Upload test file lên S3

```bash
# Từ máy local
aws s3 cp test-sample.pdf s3://arc-documents-<account>/uploads/test-001.pdf
```

### 5.2 Tạo record trong DynamoDB

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

### 5.3 Gửi message vào SQS

```bash
aws sqs send-message \
  --queue-url https://sqs.ap-southeast-1.amazonaws.com/<account>/arc-dev-document-queue \
  --message-body '{
    "doc_id": "test-001",
    "s3_key": "uploads/test-001.pdf",
    "filename": "test-sample.pdf"
  }'
```

---

## Bước 6: Monitor Processing

Xem logs của worker:

```bash
# Nếu chạy trực tiếp
# Logs hiển thị trên terminal

# Nếu chạy background
tail -f worker.log
```

Logs thành công sẽ như sau:
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

---

## Bước 7: Verify Processing

### 7.1 Kiểm tra DynamoDB

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

### 7.2 Kiểm tra Qdrant

```bash
# Trên EC2
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
---

## Xử lý Lỗi

| Vấn đề | Nguyên nhân | Giải pháp |
|--------|-------------|-----------|
| Worker không nhận message | SQS URL sai | Kiểm tra `.env` |
| Bedrock timeout | Rate limit | Tăng retry delay |
| Qdrant connection refused | Container chưa start | `docker-compose up -d qdrant` |
| FAILED status | Xem error_message trong DynamoDB | Fix và retry |

### Retry Failed Document

```bash
# Cập nhật status về UPLOADED để retry
aws dynamodb update-item \
  --table-name arc-dev-documents \
  --key '{"doc_id":{"S":"test-001"},"sk":{"S":"METADATA"}}' \
  --update-expression "SET #s = :s" \
  --expression-attribute-names '{"#s":"status"}' \
  --expression-attribute-values '{":s":{"S":"UPLOADED"}}'

# Gửi lại message vào SQS
aws sqs send-message \
  --queue-url $SQS_QUEUE_URL \
  --message-body '{"doc_id":"test-001","s3_key":"uploads/test-001.pdf","filename":"test-sample.pdf"}'
```

---

## Checklist

- [ ] Truy cập EC2 qua Session Manager
- [ ] Worker code có sẵn
- [ ] Environment variables configured
- [ ] Worker đang chạy
- [ ] Test document uploaded to S3
- [ ] SQS message sent
- [ ] Worker processed document (logs)
- [ ] Status = EMBEDDING_DONE trong DynamoDB
- [ ] Vectors stored trong Qdrant
