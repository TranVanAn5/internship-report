---
title: "Translated Blogs"
date: "`r Sys.Date()`"
weight: 3
chapter: false
pre: " <b> 3. </b> "
---



This section will list and introduce the blogs you have translated. For example:

### [Blog 1 - AWS Lambda Introduces Tiered Pricing for Amazon CloudWatch Logs and Additional Logging Targets](3.1-Blog1/)
This blog introduces AWS Lambda's new logging updates, which help serverless applications optimize costs and gain more flexibility when managing logs. The post explains that Lambda now uses tiered pricing for CloudWatch Logs, which dramatically reduces logging costs as log volumes increase. In addition to CloudWatch Logs, Lambda also supports Amazon S3 and Amazon Data Firehose as new logging targets, providing flexibility in storage, analysis, and integration with other observability platforms. The blog also covers enhancements such as advanced logging controls, CloudWatch Logs Infrequent Access, and Live Tail for increased observability. Finally, the post explains how to configure log destinations and outlines best practices for optimizing logging costs in serverless environments.
### [Blog 2 - AWS DMS Implementation Guide: Building Resilient Database Migrations Through Testing, Monitoring, and SOPs](3.2-Blog2/)
This blog is an AWS Database Migration Service (AWS DMS) implementation guide, focusing on how to build a resilient, secure, and repeatable database migration process. The content emphasizes testing before migration, monitoring during execution, and building SOPs (Standard Operating Procedures) to ensure smooth data migration, reduced risk, and easy maintenance in the AWS environment.
### [Blog 3 - AWS KMS Metrics in CloudWatch to Help You Track and Understand KMS Key Usage](3.3-Blog3/)
This blog is about the new AWS KMS feature that allows you to track API usage at the per-key level via Amazon CloudWatch, making it easy for users to see which keys are being used the most, which keys are costing money, or which keys are at risk of exceeding quota. Previously, this was quite complicated and required building custom solutions using CloudTrail and Athena, but now CloudWatch provides detailed metrics for each key, making monitoring simpler and more intuitive. The article also illustrates how to use these new metrics to query and find the KMS keys that consume the most APIs, as well as set up automatic alerts based on anomaly detection to promptly handle sudden request spikes due to configuration errors or operational processes. As a result, users can improve operational efficiency, reduce security risks, and optimize costs when using AWS KMS.