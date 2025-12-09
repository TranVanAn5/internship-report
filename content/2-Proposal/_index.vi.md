---
title: "Bản đề xuất"
date: "`r Sys.Date()`"
weight: 2
chapter: false
pre: " <b> 2. </b> "
---


# Academic Research Chatbot
## Giải pháp AWS RAG-based hỗ trợ học thuật và nghiên cứu học tập thông minh

### 1. Tóm tắt điều hành  
**Academic Research Chatbot** là trợ lý AI hỗ trợ nghiên cứu học thuật, giúp sinh viên và giảng viên tra cứu, tóm tắt và phân tích tài liệu khoa học (PDF, bài báo) thông qua hội thoại tự nhiên có trích dẫn nguồn chính xác.

**Điểm nổi bật của giải pháp:**

- **Công nghệ lõi:** Kết hợp **IDP** (Amazon Textract) để xử lý tài liệu (kể cả bản scan) và **RAG** (Amazon Bedrock - Claude 3.5 Sonnet) để sinh câu trả lời thông minh.
- **Kiến trúc tối ưu:** Mô hình Hybrid sử dụng 1 EC2 t3.small kết hợp các dịch vụ Serverless (Amplify, Cognito, S3, DynamoDB) để cân bằng hiệu năng và chi phí.
- **Tính khả thi:** Phục vụ ~50 người dùng nội bộ với chi phí vận hành **~60 USD/tháng**, thời gian triển khai nhanh (20 ngày) và tận dụng tối đa AWS Free Tier.

### 2. Tuyên bố vấn đề  
**Vấn đề hiện tại**  
Sinh viên và researcher phải làm việc với số lượng lớn tài liệu học thuật (paper hội nghị, journal, luận văn, báo cáo kỹ thuật). Nhiều tài liệu là scan PDF cũ (trước năm 2000), không có text layer, khiến việc tìm kiếm nội dung, số liệu, bảng biểu rất tốn thời gian. Các công cụ AI công cộng (ChatGPT, Perplexity, NotebookLM, v.v.) không được kết nối trực tiếp với kho tài liệu nội bộ của trường/khoa, khó đảm bảo bảo mật và quyền truy cập theo môn học hoặc nhóm nghiên cứu. Hạ tầng hiện tại không có một điểm truy cập thống nhất để:

- Quản lý tài liệu nghiên cứu theo bộ môn/đề tài.
- Cho phép researcher đặt câu hỏi trực tiếp trên chính các paper của mình.
- Đảm bảo câu trả lời có trích dẫn rõ ràng (paper, trang, bảng, mục). Hệ quả: nghiên cứu viên phải đọc thủ công, note tay, copy số liệu từ nhiều paper; giảng viên khó tổng hợp nhanh thông tin khi chuẩn bị bài giảng hoặc đề tài; dữ liệu học thuật phân tán trên nhiều máy cá nhân, khó chuẩn hóa và tái sử dụng.

**Giải pháp Academic Research Chatbot đề xuất xây dựng một nền tảng hỏi – đáp học thuật nội bộ dựa trên AWS, nơi:**  
1. **Dev/Admin nạp kho tài liệu nghiên cứu:**
   - Upload PDF vào **Amazon S3**, metadata được lưu trong **Amazon DynamoDB**.
   - Một EC2 worker tiêu thụ hàng đợi **Amazon SQS**, gọi **Amazon Textract** để OCR, trích xuất text, bảng, biểu mẫu, kể cả tài liệu scan.
   - Worker chuẩn hóa/chunk nội dung, gửi sang **Amazon Bedrock Titan Text Embeddings v2** để sinh embedding, và index vào **Qdrant** trên EC2.
2. **Researchers đặt câu hỏi qua giao diện web (Amplify + CloudFront):**
   - Câu hỏi được embed, truy vấn Qdrant để lấy các đoạn liên quan nhất (Retrieval).
   - Các đoạn này được chuyển vào **Claude 3.5 Sonnet** trên **Amazon Bedrock** để sinh câu trả lời có citation chính xác (paper, page, section, table) và giải thích theo ngữ cảnh học thuật. Toàn bộ truy cập được bảo vệ bởi **Amazon Cognito** (phân quyền researcher vs admin), log & metric được giám sát qua **Amazon CloudWatch + SNS** (cảnh báo khi có lỗi worker, queue backlog, CPU EC2 cao).

**Lợi ích và hoàn vốn đầu tư (ROI)** 
- Giảm 40–60% thời gian researcher phải bỏ ra để tìm số liệu, F1-score, p-value, sample size, thiết bị thí nghiệm hoặc mô tả phương pháp từ nhiều paper khác nhau.
- Giảm sai sót khi trích dẫn do quên trang/bảng, vì chatbot luôn trả kèm nguồn và vị trí.

**Quản lý tri thức nội bộ:**

- Tài liệu nghiên cứu được tập trung về một kho S3 + DynamoDB, dễ backup, phân quyền, và mở rộng.
- Có thể tái sử dụng cho nhiều khoá học, đề tài và lab khác nhau mà không phải xây hệ thống mới. 
  
**Chi phí hạ tầng thấp & dễ kiểm soát:**
- Mô hình hybrid 1 EC2 + managed AI services giúp chi phí vận hành cho 50 users nội bộ giữ ở mức khoảng < 50 USD/tháng, chủ yếu trả cho EC2, 2–3 VPC endpoint interface và phần sử dụng Bedrock/Textract.
- Hệ thống được thiết kế để triển khai trong khoảng 20 ngày bởi team 4 người, phù hợp làm dự án nghiên cứu/thực tập nhưng vẫn có chất lượng kiến trúc sản phẩm. 
  
**Giá trị dài hạn:**
- Tạo nền tảng để sau này tích hợp thêm dashboard phân tích hành vi học tập, module recommend paper, hoặc mở rộng sang trợ lý học tập đa ngôn ngữ và đa lĩnh vực.

### 3. Kiến trúc giải pháp  
Academic Research Chatbot áp dụng mô hình AWS Hybrid RAG Architecture với IDP (Intelligent Document Processing), kết hợp một EC2 duy nhất (FastAPI + Qdrant + Worker) với các dịch vụ AI managed (Textract, Bedrock) để vừa tối ưu chi phí, vừa đảm bảo hiệu năng cho khoảng 50 người dùng nội bộ.

Luồng xử lý dữ liệu và hội thoại  

![architecture](/images/2-Proposal/architecture.png)


*Dịch vụ AWS sử dụng*  
- **Frontend:** Route 53, CloudFront, Amplify (DNS, CDN, Host React App).
- **Auth:** Cognito (Xác thực & phân quyền researcher/admin).
- **Compute:** EC2 t3.small (FastAPI + Qdrant + Worker).
- **AI/ML:** Bedrock (Claude 3.5 Sonnet, Titan Embeddings v2).
- **IDP:** Textract (OCR cho PDF scan).
- **Storage:** S3, DynamoDB (File PDF gốc + Metadata/Status).
- **Queue:** SQS (Hàng đợi xử lý tài liệu).
- **Network:** VPC, ALB, VPC Endpoints (Bảo mật, routing, kết nối AWS Services).
- **Monitoring:** CloudWatch, SNS (Logs, Metrics, Alerts).
- **CI/CD:** CodePipeline, CodeBuild (Auto deploy backend).

**Thiết kế thành phần**  
- **Người dùng:**
    - **Researchers:** hỏi – đáp, tra cứu nội dung học thuật.
    - **Dev/Admin:** upload, quản lý và re-index tài liệu.
- **Xử lý tài liệu (IDP):**
    - PDF được Dev/Admin upload lên S3.
    - Worker trên EC2 gọi Textract để OCR và trích xuất text/bảng.
- **Lập chỉ mục (Indexing & Vector DB):**
    - Worker chuẩn hoá, chia chunk nội dung.
    - Gọi Bedrock Titan Embeddings v2 tạo embedding.
    - Lưu embedding + metadata vào Qdrant trên EC2.
- **Hội thoại AI (RAG):**
    - FastAPI embed câu hỏi, truy vấn Qdrant lấy top-k đoạn liên quan.
    - Gửi context + câu hỏi vào Claude 3.5 Sonnet (Bedrock) để sinh câu trả lời kèm citation.
- **Quản lý người dùng:**
    - Cognito xác thực và phân quyền researcher / admin.
- **Lưu trữ & trạng thái:**
    - DynamoDB lưu metadata tài liệu (doc_id, status, owner, …) và (tuỳ chọn) lịch sử chat.
### 4. Triển khai kỹ thuật  
**Các giai đoạn triển khai**  
Dự án gồm 2 phần — thiết lập trạm thời tiết biên và xây dựng nền tảng thời tiết — mỗi phần trải qua 4 giai đoạn:  
- **Nghiên cứu & chốt kiến trúc:**
    - Rà soát yêu cầu (50 researcher, 1 EC2, IDP + RAG).
    - Chốt kiến trúc VPC, EC2 (FastAPI + Qdrant + Worker), Amplify, Cognito, S3, SQS, DynamoDB, Textract, Bedrock.
- **POC & kiểm tra kết nối:**
    - Tạo EC2, VPC endpoints, thử gọi Textract, Titan Embeddings, Claude 3.5 Sonnet.
    - Chạy Qdrant đơn giản trên EC2, test insert/search vector.
    - Tạo skeleton FastAPI + một màn hình Chat UI tối giản trên Amplify.
- **Hoàn thiện tính năng chính:**
    - Xây /api/chat (FastAPI) + RAG pipeline: embed query → Qdrant → Claude + citation.
    - Xây /api/admin/: upload PDF, lưu S3 + DynamoDB, đưa message vào SQS.
    - Viết Worker trên EC2: SQS → Textract → normalize/chunk → Titan → Qdrant → update DynamoDB.
    - Hoàn thiện Chat UI và Admin UI (upload + xem trạng thái tài liệu).
- **Kiểm thử, tối ưu, triển khai demo nội bộ:**
    - Test end-to-end với một tập ~50–100 paper.
    - Thêm CloudWatch Logs/Alarms, SNS notify khi lỗi hoặc queue backlog.
    - Điều chỉnh cấu hình EC2, Qdrant, batch size để tối ưu thời gian và chi phí.
    - Chuẩn bị tài liệu hướng dẫn sử dụng và demo cho nhóm 50 researcher. 
  
**Yêu cầu kỹ thuật**

- **Frontend & Auth:**
    - React/Next.js host trên AWS Amplify, CDN CloudFront, DNS Route 53.
    - Amazon Cognito quản lý định danh và phân quyền (Researcher/Admin).
- **Backend & Compute:**
    - EC2 t3.small (Private Subnet) chạy All-in-one: FastAPI, Qdrant Vector DB và Worker.
    - Xử lý bất đồng bộ: Worker đọc SQS, kích hoạt Textract và Bedrock để index dữ liệu.
- **IDP & RAG:**
    - **Lưu trữ:** S3 (File gốc), DynamoDB (Metadata & Trạng thái).
    - **AI Core:** Textract (OCR tài liệu scan), Bedrock Titan (Embedding), Claude 3.5 Sonnet (Trả lời câu hỏi).
- **Mạng & Observability:**
    - **Network:** VPC Private Subnet, VPC Endpoints để kết nối bảo mật tới AWS Services.
    - **Monitoring:** CloudWatch Logs/Metrics + SNS cảnh báo sự cố (CPU cao, lỗi Worker).

### 5. Lộ trình & Mốc triển khai  
Dự án được thực hiện trong khoảng 6 tuần với các giai đoạn cụ thể:

- **Tuần 1-2 (Ngày 1-10): Nghiên cứu & Thiết kế**
    - Thiết kế kiến trúc chi tiết, xác định scope, dịch vụ sử dụng. Lên kế hoạch tối ưu chi phí vận hành và triển khai.
- **Tuần 3 (Ngày 11-15): Thiết lập hạ tầng AWS**
    - Cấu hình VPC, Subnets, Security Groups, IAM Roles.
    - Triển khai EC2 t3.small, S3 bucket, DynamoDB tables.
    - Thiết lập VPC Endpoints (Gateway + Interface).
- **Tuần 4 (Ngày 16-20): Backend APIs & IDP Pipeline**
    - Xây dựng FastAPI endpoints (/api/chat, /api/admin/upload).
    - Tích hợp IDP pipeline: SQS → Worker → Textract → Embeddings → Qdrant.
    - Kết nối Bedrock (Titan Embeddings + Claude 3.5 Sonnet).
- **Tuần 5 (Ngày 21-25): Testing & Error Handling**
    - Kiểm thử end-to-end với tập ~50-100 papers.
    - Xử lý edge cases, retry logic, error handling.
    - Tối ưu chunking strategy và retrieval accuracy.
- **Tuần 6 (Ngày 26-30): Deployment & Documentation**
    - Hoàn thiện UI/UX cho Admin và Researcher.
    - Thiết lập CloudWatch Alarms + SNS notifications.
    - Chuẩn bị tài liệu hướng dẫn và demo cho nhóm 50 researcher.

### 6. Ước tính ngân sách  
*Chi phí hạ tầng (ước tính theo tháng)*

- **Compute & Storage:**
    - **EC2 t3.small:** $10.08 (720h).
    - **EBS gp3:** $2.40 (30GB).
- **Network:**
    - **NAT Gateway:** $21.60.
    - **VPC Interface Endpoints:** $14.60 (2 endpoints cho Textract, Bedrock).
    - **VPC Gateway Endpoints:** FREE (S3, DynamoDB).
- **AI & Operations:**
    - **Bedrock Claude 3.5 Sonnet:** $25.00 (50 users).
    - **Bedrock Titan Embeddings:** $0.75 (750 papers).
    - **CloudWatch + Data Transfer:** $1.90.
  
*Free Tier (12 tháng đầu)*

- **Web & Auth:** S3, CloudFront, Cognito, Amplify (FREE).
- **Serverless:** DynamoDB, SQS, SNS (Always FREE).
- **IDP:** Textract AnalyzeDocument (100 pages/month trong 3 tháng đầu).
 
*Tổng cộng:* **~$60-76/tháng** (tùy mức sử dụng Bedrock).

### 7. Đánh giá rủi ro  
*Ma trận rủi ro*  
- **Hallucination (AI bịa đặt):** Ảnh hưởng cao, xác suất trung bình.
- **Vượt ngân sách (AI Services):** Ảnh hưởng trung bình, xác suất trung bình.
- **Sự cố hạ tầng (EC2/Qdrant):** Ảnh hưởng cao, xác suất thấp.

*Chiến lược giảm thiểu*  
- **Chất lượng AI:** Bắt buộc trích dẫn nguồn (citation), giới hạn context đầu vào từ Qdrant.
- **Chi phí:** Thiết lập AWS Budgets/Alarms, kiểm soát số lượng tài liệu ingest.
- **Hạ tầng & Bảo mật:** Backup EBS định kỳ, mã hóa dữ liệu (S3/DynamoDB), phân quyền chặt chẽ qua Cognito/IAM.

*Kế hoạch dự phòng*  
- **Sự cố hệ thống:** Khôi phục từ Snapshot, tạm dừng ingestion (buffer qua SQS).
- **Vượt chi phí:** Tạm khóa tính năng upload mới, giới hạn hạn ngạch truy vấn trong ngày.

### 8. Kết quả kỳ vọng  
*Cải tiến kỹ thuật*: 

- Chuyển đổi kho tài liệu rời rạc (PDF/Scan) thành tri thức số có thể truy vấn và trích dẫn tự động.
- Giảm đáng kể thời gian tra cứu thủ công nhờ công nghệ RAG + IDP. Giá trị dài hạn
- Xây dựng nền tảng nghiên cứu số hóa cho 50+ researcher, dễ dàng mở rộng quy mô.
- Tạo tiền đề phát triển các tính năng nâng cao: Gợi ý tài liệu, phân tích xu hướng nghiên cứu và hỗ trợ viết tổng quan (Literature Review).