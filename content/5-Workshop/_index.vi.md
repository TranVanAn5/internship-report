---
title : "Workshop"
date :  "`r Sys.Date()`" 
weight : 5
chapter : false
pre : " <b> 5. </b> "
---

#### Tổng quan

Trong workshop này, chúng ta sẽ xây dựng **ARC (Academic Research Chatbot)** - một hệ thống chatbot thông minh hoạt động trên nền tảng **AWS Serverless**. Giải pháp này ứng dụng **Generative AI** và **RAG (Retrieval-Augmented Generation)** để hỗ trợ nghiên cứu học thuật, truy vấn tài liệu và trả lời câu hỏi một cách linh hoạt.

Thay vì trả lời các câu hỏi dựa trên kịch bản cố định (rule-based), hệ thống sử dụng mô hình **Claude 3.5 Sonnet** để hiểu ngôn ngữ tự nhiên, truy vấn dữ liệu từ cơ sở vector database và phản hồi người dùng một cách chính xác.

#### Mục tiêu Workshop

Sau khi hoàn thành workshop, bạn sẽ:

- Hiểu kiến trúc RAG và cách áp dụng vào thực tế  
- Triển khai hệ thống chatbot hoàn chỉnh trên AWS  
- Sử dụng Amazon Bedrock (Claude 3.5 Sonnet + Cohere Embed)  
- Xây dựng IDP pipeline với Amazon Textract  
- Implement vector search với Qdrant  
- Deploy infrastructure với Terraform  
- Tích hợp authentication với Amazon Cognito  

#### Nội dung

1. [Giới thiệu](1-introduction/)
2. [Các bước chuẩn bị](2-preparation/)
3. [Kích hoạt Bedrock Models](3-bedrock-models/)
4. [Cấu hình AWS CLI](4-aws-cli/)
5. [Chuẩn bị Dữ liệu](5-data-preparation/)
6. [Triển khai Infrastructure](6-infrastructure/)
7. [Thiết lập Backend API](7-backend/)
8. [Thiết lập IDP Pipeline](8-idp-pipeline/)
9. [Thiết lập Frontend](9-frontend/)
10. [Sử dụng Chatbot](10-using-chatbot/)
11. [Sử dụng Admin Dashboard](11-admin-dashboard/)
12. [Dọn dẹp Tài nguyên](12-cleanup/)
