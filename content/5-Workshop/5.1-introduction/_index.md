---
title : "Introduction"
date :  "`r Sys.Date()`" 
weight : 1
chapter : false
pre : " <b> 5.1 </b> "
---

## Problem Statement

Traditional chatbot systems struggle without the ability to access specific information from internal documents, leading to inaccurate or irrelevant responses. This workshop solves this problem by building an architecture capable of:

- **Automation**: Automatically process and index PDF documents
- **Content Inquiry**: Receive queries and guide users to relevant content
- **Document Retrieval**: Answer complex questions with accurate source citations from documents

## Solution Architecture

The system is designed following the **RAG (Retrieval-Augmented Generation)** model combined with **AWS Serverless** to ensure scalability:

1. **Frontend Interface**: Users interact via React Web Application
   - Amazon API Gateway receives requests from Frontend
   - AWS Amplify hosting with CloudFront CDN
   - Amazon Cognito handles authentication

2. **Request Handling**: 
   - Application Load Balancer routes traffic to EC2
   - FastAPI Backend processes REST API requests
   - Amazon SQS (FIFO) ensures document processing order

3. **Backend Processing**:
   - ChatHandler: Manages conversations, saves sessions to Amazon DynamoDB
   - RAG Service: Orchestrates vector search and LLM generation
   - Qdrant Vector Database: Self-hosted on EC2 for vector search

4. **AI & Data Layer**:
   - Amazon Bedrock: Uses Claude 3.5 Sonnet (LLM) and Cohere Embed Multilingual v3 (Embeddings)
   - Amazon Textract: OCR and text extraction from PDF
   - Amazon S3: Document storage
   - Amazon DynamoDB: Metadata and chat history

5. **Admin Dashboard**: 
   - React-based interface hosted on AWS Amplify
   - Upload and manage documents
   - Monitor processing status
   - View chat history

## Architecture

![architech](/images/2-Proposal/FCJ-MVP.png)

## Key Technologies

In this workshop, you will work with the following key AWS services:

- **Amazon Bedrock**: The heart of AI, providing Foundation Models (Claude, Cohere) for language processing and embedding generation
- **Amazon Textract**: Build IDP pipeline to extract text from PDF documents
- **Amazon EC2 & VPC**: Compute and network infrastructure for backend services
- **Amazon S3**: Document and static asset storage
- **Amazon DynamoDB**: Store metadata, chat history, and document status
- **Amazon Cognito**: Authentication and user management
- **AWS Amplify**: Frontend application hosting with integrated CI/CD
- **Amazon SQS**: Message queue for document processing pipeline
- **Qdrant (Self-hosted)**: Vector database for semantic search
- **Terraform (IaC)**: Deploy entire infrastructure as code