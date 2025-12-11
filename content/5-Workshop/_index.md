---
title: "Workshop"
date: "`r Sys.Date()`"
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# AI-POWERED ACADEMIC RESEARCH CHATBOT ON AWS

#### Overview

In this workshop, we will build **ARC (Academic Research Chatbot)** - an intelligent chatbot system running on the **AWS Serverless** platform. This solution leverages **Generative AI** and **RAG (Retrieval-Augmented Generation)** to support academic research, document queries, and answer questions flexibly.

Instead of answering questions based on fixed scripts (rule-based), the system uses the **Claude 3.5 Sonnet** model to understand natural language, query data from the vector database, and respond to users accurately.

#### Workshop Objectives

After completing this workshop, you will:

- Understand RAG architecture and how to apply it in practice  
- Deploy a complete chatbot system on AWS  
- Use Amazon Bedrock (Claude 3.5 Sonnet + Cohere Embed)  
- Build an IDP pipeline with Amazon Textract  
- Implement vector search with Qdrant  
- Deploy infrastructure with Terraform  
- Integrate authentication with Amazon Cognito  

#### Content

1. [Introduction](1-introduction/)
2. [Preparation](2-preparation/)
3. [Activate Bedrock Models](3-bedrock-models/)
4. [Configure AWS CLI](4-aws-cli/)
5. [Data Preparation](5-data-preparation/)
6. [Deploy Infrastructure](6-infrastructure/)
7. [Setup Backend API](7-backend/)
8. [Setup IDP Pipeline](8-idp-pipeline/)
9. [Setup Frontend](9-frontend/)
10. [Using Chatbot](10-using-chatbot/)
11. [Using Admin Dashboard](11-admin-dashboard/)
12. [Clean up Resources](12-cleanup/)