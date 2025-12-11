---
title : "Giới thiệu"
date :  "`r Sys.Date()`" 
weight : 1
chapter : false
pre : " <b> 5.1 </b> "
---

## Tuyên bố vấn đề

Các hệ thống chatbot truyền thống gặp khó khăn khi không có khả năng truy cập thông tin cụ thể từ tài liệu nội bộ, dẫn đến trả lời không chính xác hoặc không liên quan. Workshop này giải quyết vấn đề bằng cách xây dựng một kiến trúc có khả năng:

- **Tự động hóa**: Xử lý và đánh chỉ mục tài liệu PDF tự động
- **Hỏi nội dung**: Nhận truy vấn và điều hướng người dùng đến nội dung liên quan
- **Truy xuất tài liệu**: Trả lời các câu hỏi phức tạp với trích dẫn nguồn chính xác từ tài liệu

## Kiến trúc giải pháp

Hệ thống được thiết kế theo mô hình **RAG (Retrieval-Augmented Generation)** kết hợp với **AWS Serverless** để đảm bảo khả năng mở rộng:

1. **Frontend Interface**: Người dùng tương tác qua React Web Application
   - Amazon API Gateway nhận requests từ Frontend
   - AWS Amplify hosting với CloudFront CDN
   - Amazon Cognito xử lý authentication

2. **Request Handling**: 
   - Application Load Balancer định tuyến traffic đến EC2
   - FastAPI Backend xử lý REST API requests
   - Amazon SQS (FIFO) đảm bảo thứ tự xử lý documents

3. **Backend Processing**:
   - ChatHandler: Quản lý hội thoại, lưu session vào Amazon DynamoDB
   - RAG Service: Orchestrate vector search và LLM generation
   - Qdrant Vector Database: Self-hosted trên EC2 cho vector search

4. **AI & Data Layer**:
   - Amazon Bedrock: Sử dụng Claude 3.5 Sonnet (LLM) và Cohere Embed Multilingual v3 (Embeddings)
   - Amazon Textract: OCR và trích xuất text từ PDF
   - Amazon S3: Lưu trữ documents
   - Amazon DynamoDB: Metadata và chat history

5. **Admin Dashboard**: 
   - React-based interface hosted trên AWS Amplify
   - Upload và quản lý documents
   - Monitor processing status
   - View chat history

## Architect

![architech](/images/2-Proposal/FCJ-MVP.png)

## Key Technologies

Trong workshop này, bạn sẽ làm việc với các dịch vụ AWS chính sau:

- **Amazon Bedrock**: Trái tim của AI, cung cấp các Foundation Models (Claude, Cohere) để xử lý ngôn ngữ và sinh embeddings
- **Amazon Textract**: Xây dựng IDP pipeline để trích xuất text từ PDF documents
- **Amazon EC2 & VPC**: Cơ sở hạ tầng compute và network cho backend services
- **Amazon S3**: Lưu trữ documents và static assets
- **Amazon DynamoDB**: Lưu trữ metadata, chat history và document status
- **Amazon Cognito**: Authentication và user management
- **AWS Amplify**: Hosting frontend application với CI/CD tích hợp
- **Amazon SQS**: Message queue cho document processing pipeline
- **Qdrant (Self-hosted)**: Vector database cho semantic search
- **Terraform (IaC)**: Triển khai toàn bộ hạ tầng dưới dạng mã

