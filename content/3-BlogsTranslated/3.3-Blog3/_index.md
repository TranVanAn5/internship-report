---
title: "Blog 3"
date: "`r Sys.Date()`"
weight: 1
chapter: false
pre: " <b> 3.3. </b> "
---

# AWS KMS metrics in CloudWatch help you monitor and better understand KMS key usage

By Norman Li and Haiyu Zhen | March 17, 2025 | in [AWS Key Management Service](https://aws.amazon.com/blogs/security/category/security-identity-compliance/aws-key-management-service/), [Best Practices](https://aws.amazon.com/blogs/security/category/post-types/best-practices/), [Intermediate (200)](https://aws.amazon.com/blogs/security/category/learning-levels/intermediate-200/), [Security, Identity, & Compliance](https://aws.amazon.com/blogs/security/category/security-identity-compliance/), [Technical How-to](https://aws.amazon.com/blogs/security/category/post-types/technical-how-to/) |

[AWS Key Management Service (AWS KMS)](https://aws.amazon.com/kms/) is pleased to introduce key-level filtering for AWS KMS API usage in [Amazon CloudWatch](https://aws.amazon.com/cloudwatch/) metrics, providing enhanced visibility to help customers improve operational efficiency and support security and compliance risk management.

AWS KMS now publishes account-level AWS KMS API usage metrics [with Amazon CloudWatch](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch-Service-Quota-Integration.html), allowing you to track and manage API usage. However, if you are using multiple KMS keys, it can be difficult to determine exactly which keys are using the most request rate quota or causing significant API costs. For example, if you have more than 10 active KMS keys in your account, prior to this feature launch, you would need to build a custom solution based on CloudTrail and Amazon Athena to identify which keys are driving the majority of your API usage and costs. With the new CloudWatch metrics available in the AWS/KMS namespace in CloudWatch, you can monitor, understand, and alarm on granular API usage at the individual KMS key level without building a costly custom solution.

This blog post explores several use cases that help you better leverage these newly introduced CloudWatch metrics to manage AWS KMS API usage and costs. Use cases include viewing and understanding your API usage at the key level, as well as creating CloudWatch alarms to detect unintended overage.

## Overview of New CloudWatch Metrics for KMS Keys

With CloudWatch metrics for KMS keys, you can now do the following:

1. View API usage for a specific KMS key, filtered by each API operation (e.g., Encrypt, Decrypt, or GenerateDataKey).

2. View aggregate usage across encryption operations for a given KMS key.

3. Set up alerts if a specific KMS key exceeds a specified threshold on a single API operation or a set of API operations.

This streamlined approach allows you to quickly monitor, understand, and troubleshoot KMS key API usage patterns without the multi-step process that was previously required. Let's dive into how to use these key-level API usage metrics in two real-world examples.

### Example 1: How to Identify KMS Keys Consuming the Most API Quota or Contributing the Most API Charges

When you exceed your AWS KMS API request quota, you can view your AWS KMS API usage in the [Service Quotas console.](https://console.aws.amazon.com/servicequotas) However, you may still have difficulty identifying the KMS keys that are taking up the most request quota. When you receive an excess AWS KMS API charge, you can check the detailed usage in each AWS Region in Cost Explorer, but you cannot easily find the KMS keys with the highest API charges. This process becomes even more difficult when you manage a large number of KMS keys.

With CloudWatch key-level API usage metrics, you can use the advanced metrics query option to query CloudWatch Metrics Insights data in user-friendly SQL to identify the KMS keys that account for the majority of API usage quota or contribute the most API charges.

#### Walkthrough

To use Amazon CloudWatch Metrics Insights to identify the top 20 KMS keys with the most encryption API usage over the past 3 hours, complete the following steps:

1. Open the [CloudWatch console](https://console.aws.amazon.com/cloudwatch/).
2. In the navigation pane, select **Metrics**, then select **All metrics**.
3. Select the **Multi source query** tab.
4. For the data source, select **CloudWatch Metrics Insights**.

5. You can enter the following example query in Editor view:

Note: In Builder view, the metric namespace, metric name, filter by, group by, sort by, and limit options are displayed. In Editor view, the same options as in Builder view are displayed in query format.

| |
| :---: |
| |
| SELECT SUM(SuccessfulRequest) |
| FROM SCHEMA(\"AWS/KMS\", KeyArn, Operation) |
| GROUP BY KeyArn |
| ORDER BY MAX () DESC |
| LIMIT 20 |
| |

6. Select **Run** in Editor view or **Graph query** in Builder view.

### Example 2: How to set up new granular alerts for unintended AWS KMS API usage

Running big data workflows that read [Amazon Simple Storage Service (Amazon S3)](https://aws.amazon.com/s3/) files encrypted with KMS keys is a common scenario for analytics, business reporting, or machine learning projects. Typically, these processes only read a limited number of files from S3 per call. However, misconfigured processes can accidentally read a large number of S3 files, leading to exceeding your AWS KMS API request rate quotas or incurring unexpected charges due to spiky AWS KMS API usage. Previously, to address this issue, you would have to build a custom alerting system by following these steps: 1) sending AWS CloudTrail events generated by AWS KMS to Amazon CloudWatch Logs; 2) writing queries in Amazon CloudWatch Logs Insights to track your API request usage; and 3) enabling anomaly detection on the corresponding CloudWatch Log Insights mathematical expression.

Now, with CloudWatch API usage metrics at the key level, you can enable anomaly detection directly on these metrics to set up alerts for unusual AWS KMS API usage patterns. This provides a more streamlined and effective way to monitor and detect potentially risky workflows. Using CloudWatch metrics and this anomaly detection capability, you can proactively identify and address unintended increases in AWS KMS API usage, helping to avoid unexpected charges or service disruptions in your analytics, reporting, or machine learning workflows.

#### Walkthrough

Consider a scenario where you have an analytics workflow that runs regularly, using the **Decrypt** AWS KMS API on a KMS key to decrypt and read data from S3. You want to enable anomaly detection on the KMS key to trigger an alarm when a **Decrypt** call to a specific KMS key detects a clear trend or pattern. To do this, complete the following steps:

1. Open the [CloudWatch console](https://console.aws.amazon.com/cloudwatch/).

2. In the navigation pane, select **Metrics**, then select **All metrics**.

3. Select **KMS**, then select **KeyArn**, **Operation**.

4. In the search bar, enter the Amazon Resource Name (**ARN**) of the key, then select **Search**. Select the CloudWatch metric for which you want to enable anomaly detection.

5. Navigate to **Graphed metrics**, and using the **Statistic** and **Period** drop-down lists, select the statistic and period you want to monitor. You can then enable anomaly detection by selecting the Pulse icon.

6. You can [tune anomaly detection](https://aws.amazon.com/blogs/mt/operationalizing-cloudwatch-anomaly-detection/) by setting the sensitivity to adjust bandwidth, if needed.

### Conclusion

This blog post highlighted the newly introduced key-level filtering capabilities for AWS KMS API usage in CloudWatch. We presented two real-world use cases to illustrate how you can use the new CloudWatch metrics. These use cases include improving operational visibility, setting up proactive alerts for anomalies in KMS API usage patterns, and being able to track granular key usage for compliance purposes.

If you have feedback on this article, please post it in the Comments section below. If you have questions about this article, please create a new topic in [AWS Key Management Service re:Post](https://forums.aws.amazon.com/forum.jspa?forumID=182).

-----

| |
| :---: |
| **Norman Li**Norman is the Director of Software Development for AWS KMS. In this role, Norman leads the development of visibility features, as well as internal scaling initiatives. Outside of work, Norman enjoys spending time in the beautiful mountains of the Pacific Northwest. |

| |
| :---: |
| **Haiyu Zhen**Haiyu is a Senior Software Development Engineer for AWS KMS. She specializes in building secure, large-scale distributed systems and is passionate about enhancing cloud-native application security without compromising performance. |

TAGS: [AWS Key Management Service](https://aws.amazon.com/blogs/security/tag/aws-key-management-service/), [AWS Key Management Service (KMS)](https://aws.amazon.com/blogs/security/tag/aws-key-management-service-kms/), [Security Blog](https://aws.amazon.com/blogs/security/tag/security-blog/)