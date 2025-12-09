---
title: "Proposal"
date: "`r Sys.Date()`"
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# IoT Weather Platform for Lab Research
## A Unified AWS Serverless Solution for Real-Time Weather Monitoring

### 1. Executive Summary
**Academic Research Chatbot** is an AI assistant that supports academic research, helping students and lecturers to look up, summarize and analyze scientific documents (PDFs, articles) through natural conversations with accurate source citations.

**Solution Highlights:**

- **Core Technology:** Combining **IDP** (Amazon Textract) to process documents (including scans) and **RAG** ​​(Amazon Bedrock - Claude 3.5 Sonnet) to generate intelligent answers.

- **Optimized Architecture:** Hybrid model using 1 EC2 t3.small combined with Serverless services (Amplify, Cognito, S3, DynamoDB) to balance performance and cost.

- **Feasibility:** Serving ~50 internal users with operating costs of **~60 USD/month**, fast deployment time (20 days) and making the most of AWS Free Tier.

### 2. Problem Statement
### What’s the Problem?
Students and researchers have to work with a large number of academic documents (conference papers, journals, theses, technical reports). Many documents are old PDF scans (before 2000), without text layers, making searching for content, figures, and tables very time-consuming. Public AI tools (ChatGPT, Perplexity, NotebookLM, etc.) are not directly connected to the internal document repository of the school/faculty, making it difficult to ensure security and access rights by subject or research group. The current infrastructure does not have a unified access point to:

- Manage research documents by subject/topic.
- Allow researchers to ask questions directly on their own papers.
- Ensure answers have clear citations (paper, page, table, section). Consequences: researchers have to read manually, take notes by hand, copy figures from many papers; lecturers have difficulty quickly synthesizing information when preparing lectures or topics; Academic data is distributed across many personal machines, making it difficult to standardize and reuse.

### The Solution
1. **Dev/Admin loads research document repository:**
- Upload PDF to **Amazon S3**, metadata is stored in **Amazon DynamoDB**.
- An EC2 worker consumes **Amazon SQS** queue, calls **Amazon Textract** to OCR, extracts text, tables, forms, including scanned documents.
- Worker normalizes/chunks content, sends to **Amazon Bedrock Titan Text Embeddings v2** to generate embedding, and indexes into **Qdrant** on EC2.
2. **Researchers ask questions via web interface (Amplify + CloudFront):**
- Question is embedded, query Qdrant to get the most relevant paragraphs (Retrieval).
- These paragraphs are passed to **Claude 3.5 Sonnet** on **Amazon Bedrock** to generate answers with correct citation (paper, page, section, table) and explanation in academic context. All access is protected by **Amazon Cognito** (researcher vs admin authorization), logs & metrics are monitored via **Amazon CloudWatch + SNS** (alerts for worker errors, queue backlog, high EC2 CPU).

### Benefits and Return on Investment
- Reduce 40–60% of the time researchers have to spend to find data, F1-score, p-value, sample size, experimental equipment or method description from many different papers.

- Reduce errors when citing due to forgetting pages/tables, because the chatbot always returns the source and location.

**Internal knowledge management:**

- Research documents are centralized in an S3 + DynamoDB repository, easy to backup, decentralize, and expand.

- Can be reused for many different courses, topics and labs without having to build a new system.

**Low infrastructure cost & easy to control:**
- The hybrid model of 1 EC2 + managed AI services helps keep operating costs for 50 internal users at around <50 USD/month, mainly paying for EC2, 2–3 VPC endpoint interfaces and Bedrock/Textract usage.

- The system is designed to be deployed in about 20 days by a team of 4 people, suitable for research/internship projects but still has product architecture quality.

**Long-term value:**
- Create a foundation to later integrate a learning behavior analysis dashboard, a recommendation paper module, or expand to a multilingual and multidisciplinary learning assistant.

### 3. Solution Architecture
Academic Research Chatbot applies the AWS Hybrid RAG Architecture model with IDP (Intelligent Document Processing), combining a single EC2 (FastAPI + Qdrant + Worker) with managed AI services (Textract, Bedrock) to optimize costs while ensuring performance for about 50 internal users.

Data processing and conversation flow

![architecture](/images/2-Proposal/architecture.png)

*AWS services used*
- **Frontend:** Route 53, CloudFront, Amplify (DNS, CDN, Host React App).
- **Auth:** Cognito (Authentication & authorization of researcher/admin).
- **Compute:** EC2 t3.small (FastAPI + Qdrant + Worker).
- **AI/ML:** Bedrock (Claude 3.5 Sonnet, Titan Embeddings v2).

- **IDP:** Textract (OCR for PDF scans).
- **Storage:** S3, DynamoDB (Original PDF file + Metadata/Status).
- **Queue:** SQS (Document processing queue).
- **Network:** VPC, ALB, VPC Endpoints (Security, routing, AWS Services connection).
- **Monitoring:** CloudWatch, SNS (Logs, Metrics, Alerts).
- **CI/CD:** CodePipeline, CodeBuild (Auto deploy backend).

**Component design**
- **Users:**
   - **Researchers:** Q&A, academic content lookup.
   - **Dev/Admin:** upload, manage and re-index documents.
- **Document processing (IDP):**
   - PDF is uploaded to S3 by Dev/Admin.
   - Worker on EC2 calls Textract to OCR and extract text/tables.
- **Indexing & Vector DB:**
   - Worker normalizes, chunks content.
   - Call Bedrock Titan Embeddings v2 to create embedding.
   - Save embedding + metadata to Qdrant on EC2.
- **AI Conversation (RAG):**
   - FastAPI embeds question, queries Qdrant to get top-k related segments.
   - Send context + question to Claude 3.5 Sonnet (Bedrock) to generate answers with citations.
- **User Management:**
   - Cognito authenticates and authorizes researcher / admin.
- **Storage & State:**
   - DynamoDB stores document metadata (doc_id, status, owner, …) and (optionally) chat history.

### 4. Technical Implementation
**Implementation Phases**
The project consists of 2 parts — setting up the edge weather station and building the weather platform — each part goes through 4 phases:
- **Research & finalize the architecture:**
   - Review requirements (50 researchers, 1 EC2, IDP + RAG).
   - Finalize the architecture of VPC, EC2 (FastAPI + Qdrant + Worker), Amplify, Cognito, S3, SQS, DynamoDB, Textract, Bedrock.
- **POC & test connectivity:**
   - Create EC2, VPC endpoints, test calling Textract, Titan Embeddings, Claude 3.5 Sonnet.
   - Run simple Qdrant on EC2, test insert/search vector.
   - Create FastAPI skeleton + a minimalist Chat UI screen on Amplify.
- **Complete main features:**
   - Build /api/chat (FastAPI) + RAG pipeline: embed query → Qdrant → Claude + citation.
   - Build /api/admin/: upload PDF, save S3 + DynamoDB, put message into SQS.
   - Write Worker on EC2: SQS → Textract → normalize/chunk → Titan → Qdrant → update DynamoDB.
   - Complete Chat UI and Admin UI (upload + view document status).
- **Test, optimize, deploy internal demo:**
   - End-to-end test with a set of ~50–100 papers.
   - Add CloudWatch Logs/Alarms, SNS notify when error or queue backlog.
   - Adjust EC2 configuration, Qdrant, batch size to optimize time and cost.
   - Prepare user manual and demo for group of 50 researchers.

**Technical requirements**

- **Frontend & Auth:**
   - React/Next.js hosted on AWS Amplify, CDN CloudFront, DNS Route 53.
   - Amazon Cognito for identity and authorization management (Researcher/Admin).

- **Backend & Compute:**
   - EC2 t3.small (Private Subnet) running All-in-one: FastAPI, Qdrant Vector DB and Worker.
   - Asynchronous processing: Worker reads SQS, triggers Textract and Bedrock to index data.

- **IDP & RAG:**
   - **Storage:** S3 (Original file), DynamoDB (Metadata & State).
   - **AI Core:** Textract (OCR scanned documents), Bedrock Titan (Embedding), Claude 3.5 Sonnet (Question Answering).

- **Network & Observability:**
   - **Network:** VPC Private Subnet, VPC Endpoints for secure connection to AWS Services.
   - **Monitoring:** CloudWatch Logs/Metrics + SNS incident alert (high CPU, Worker error).

### 5. Timeline & Milestones
*The project was implemented in about 6 weeks with specific stages:

- **Week 1-2 (Days 1-10): Research & Design**
   - Design detailed architecture, determine scope, services used. Plan to optimize operating and deployment costs.
- **Week 3 (Days 11-15): Set up AWS infrastructure**
   - Configure VPC, Subnets, Security Groups, IAM Roles.
   - Deploy EC2 t3.small, S3 bucket, DynamoDB tables.
   - Set up VPC Endpoints (Gateway + Interface).
- **Week 4 (Days 16-20): Backend APIs & IDP Pipeline**
   - Build FastAPI endpoints (/api/chat, /api/admin/upload).
   - Integrate IDP pipeline: SQS → Worker → Textract → Embeddings → Qdrant.
   - Bedrock Connectivity (Titan Embeddings + Claude 3.5 Sonnet).
- **Week 5 (Days 21-25): Testing & Error Handling**
   - End-to-end testing with ~50-100 papers.
   - Handling edge cases, retry logic, error handling.
   - Optimizing chunking strategy and retrieval accuracy.
- **Week 6 (Days 26-30): Deployment & Documentation**
   - Finalizing UI/UX for Admin and Researcher.
   - Setting up CloudWatch Alarms + SNS notifications.
   - Preparing documentation and demo for a group of 50 researchers.

### 6. Budget Estimation
*Infrastructure costs (estimated by month)*

- **Compute & Storage:** 
- **EC2 t3.small:** $10.08 (720h). 
- **EBS gp3:** $2.40 (30GB).
- **Network:** 
- **NAT Gateway:** $21.60. 
- **VPC Interface Endpoints:** $14.60 (2 endpoints for Textract, Bedrock). 
- **VPC Gateway Endpoints:** FREE (S3, DynamoDB).
- **AI & Operations:** 
- **Bedrock Claude 3.5 Sonnet:** $25.00 (50 users). 
- **Bedrock Titan Embeddings:** $0.75 (750 papers). 
- **CloudWatch + Data Transfer:** $1.90.

*Free Tier (first 12 months)*

- **Web & Auth:** S3, CloudFront, Cognito, Amplify (FREE).

- **Serverless:** DynamoDB, SQS, SNS (Always FREE).

- **IDP:** Textract AnalyzeDocument (100 pages/month for the first 3 months).

**Total:** **~$60-76/month** (depending on Bedrock usage).


### 7. Risk Assessment
*Risk Matrix*

- **Hallucination (fabricated AI):** High impact, medium probability.
- **Budget Overrun (AI Services):** Medium impact, medium probability.
- **Infrastructure Incident (EC2/Qdrant):** High impact, low probability.

*Mitigation Strategy*

- **AI Quality:** Required citation, limited input context from Qdrant.
- **Cost:** Set up AWS Budgets/Alarms, control the number of ingest documents.
- **Infrastructure & Security:** Regular EBS backup, data encryption (S3/DynamoDB), strict authorization via Cognito/IAM.

*Contingency Plan*

- **System Incident:** Restore from Snapshot, pause ingestion (buffer via SQS).
- **Cost overrun:** Temporarily lock new upload feature, limit daily query quota.

### 8. Expected Outcomes
*Technical improvements*:

- Converting discrete document repositories (PDF/Scan) into digital knowledge that can be automatically queried and cited.
- Significantly reducing manual search time thanks to RAG + IDP technology. Long-term value
- Building a digital research platform for 50+ researchers, easy to scale.
- Creating the premise for developing advanced features: Suggesting documents, analyzing research trends and supporting literature reviews.