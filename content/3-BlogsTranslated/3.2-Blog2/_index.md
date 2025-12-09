---
title: "Blog 2"
date: "`r Sys.Date()`"
weight: 1
chapter: false
pre: " <b> 3.2. </b> "
---

# AWS DMS Deployment Guide: Building Resilient Database Migrations Through Testing, Monitoring, and SOPs

By Sushant Deshmukh, Alex Anto Kizhakeyyepunnil Joy, and Sanyam Jain | May 5, 2025 | in [Advanced (300)](https://aws.amazon.com/blogs/database/category/learning-levels/advanced-300/), [AWS Database Migration Service (AWS DMS)](https://aws.amazon.com/blogs/database/category/database/aws-database-migration-service/), [AWS Well-Architected](https://aws.amazon.com/blogs/database/category/aws-well-architected/),[Best Practices](https://aws.amazon.com/blogs/database/category/post-types/best-practices/) |

[AWS Database Migration Service](https://aws.amazon.com/dms/) (AWS DMS) simplifies database migration and replication, providing a managed solution for customers. Our observations across multiple enterprise deployments show that investing time in proactive database migration planning pays off. Organizations that adopt a comprehensive setup strategy consistently experience fewer disruptions and better migration outcomes.

In this article, we outline proactive measures to optimize AWS DMS deployments from the initial setup phase. By using strategic planning and architectural vision, organizations can improve the reliability of their replication systems, improve performance, and avoid common pitfalls.

We explore strategies and best practices in the following key areas:

1. Plan and run proof-of-concept (PoC) tests
2. Implement system failure testing
3. Develop standard operating procedures (SOPs)
4. Monitor and alert
5. Apply the principles of the [AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html)

## Plan and execute PoCs

Executing PoCs helps detect and fix environmental issues early. It also generates information that you can use to estimate your total migration time and resource commitments.

Here are the basic steps to a successful PoC:

1. Plan and deploy a test environment with the appropriate replicated instances, tasks, and AWS DMS endpoints. For more information on resource planning and provisioning, you can refer to [AWS Database Migration Service (AWS DMS) Best Practices].](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_BestPractices.html)
2. Use workloads similar to those in the production environment. It is imperative to simulate your production environment as closely as possible to increase the likelihood of encountering various issues.
3. Perform error testing based on the scenarios discussed in the table in the next section.
4. Monitor resource utilization and bottlenecks that occur in the PoC and review the resource planning and deployment accordingly.
5. Record your observations and perform migration evaluation by comparing them with business outcomes. This includes evaluating post-migration recovery times and application Service Level Agreements (SLAs) for both the migration and ongoing business operations. If these migration and operational requirements are not met, revisit the planning phase to ensure alignment with your business needs.

## Conduct systematic failure testing

All systems, regardless of their resilience, are subject to failures and downtime. For organizations operating mission-critical workloads, proactive planning is essential to maintaining business continuity and meeting SLAs. This section provides a strategic framework for developing Standard Operating Procedures (SOPs) that establish clear recovery protocols and minimize operational impact in the event of a system disruption.

When implementing AWS DMS, understanding potential points of failure becomes critical to building resilient systems. The following table outlines common failure scenarios in AWS DMS replication systems, which should serve as a foundation for your testing strategy. While this table is comprehensive, we encourage you to expand on these scenarios based on your specific architecture, compliance requirements, and business objectives to fully cover the potential failure modes in your environment.

| Point of Failure | Potential Downtime Scenarios | Testing | Potential Mitigation Strategies |
| :---: | :---: | :---: | :---: |
| Source and Target Databases | Performance bottlenecks on the database server such as high CPU, memory constraints. | Stress your system with a benchmark tool like [sysbench](https://aws.amazon.com/blogs/database/running-sysbench-on-rds-mysql-rds-mariadb-and-amazon-aurora-mysql-via-ssl-tls/) to simulate high load on the database server. | You can provide at Read-only database nodes for AWS DMS-supported engines, using a read replica as the source. For more information, see [Data Migration Source](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Source.html). You can also scale database resources and optimize database parameters. |
| | Data access issues due to insufficient permissions | Using a database account for AWS DMS that does not have sufficient permissions. | Create a database user using the least-privilege principle. Refer to the [documentation](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Source.html) for the corresponding AWS DMS source endpoint for required DMS privileges for each database engine. |
| | Failover database (if using a primary or standby setup) | Perform a database failover from the primary node to the secondary node. | In case AWS DMS may attempt to reconnect to the old primary after failover, this behavior depends on the Time to Live (TTL). You will need to restart the task after the TTL is refreshed. See [Why is my connection being redirected to the reader instance when I try to connect to my Amazon Aurora writer endpoint?](https://repost.aws/knowledge-center/aurora-mysql-redirected-endpoint-writer) |
| | Shut down the database OR a failure occurs | Stop the database that is replicating DMS. | Record all DMS task behavior for SOP development, then resume the task when the database issue has been resolved. |
| | Transaction logs not available | Delete shorter retention logs when the task is offline or running slow. | Log DMS task observations to generate SOP and continue the task after generating the transaction log or performing a new full load if the log is not available. |
| | Make structural changes such as schemas, tables, indexes, partitions, and data types | Run various data definition language (DDL) statements to modify the relevant table. | See the list of [supported DDL statements](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Introduction.SupportedDDL.html) and [task settings.](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Tasks.CustomizingTasks.TaskSettings.DDLHandling.html) |
| Network errors (applies to source and destination) | Connectivity issues including network, DNS, and SSL errors | Remove the source IP from the AWS DMS security group OR modify iptables; Remove the certificate from the DMS endpoint; Modify the MTU (maximum transmission unit) value. | Refer to troubleshooting [AWS DMS endpoint connectivity](https://repost.aws/knowledge-center/dms-endpoint-connectivity-failures) and [problems connecting to Amazon RDS.](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Troubleshooting.html#CHAP_Troubleshooting.General.RDSConnection) |
| | Packet loss | Use the traffic control (tc) command on Linux systems OR use the AWS Fault Injection Simulator (FIS). | See [Troubleshooting Networking Issues During Database Migration with the AWS DMS Diagnostic Support AMI](https://aws.amazon.com/blogs/database/troubleshoot-networking-issues-during-database-migration-with-the-aws-dms-diagnostic-support-ami/) and [Working with the AWS DMS Diagnostic Support AMI](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_SupportAmi.html) |
| AWS DMS Error | Restart Single-AZ Replication Instance error | Restart AWS DMS replication instance. | DMS will automatically resume tasks after the replication instance restarts. |
| | Restart Multi-AZ Replication instance with failover during replication in progress | Restart AWS DMS replication instance, selecting the “Restart with planned failover?” option. | DMS automatically resumes tasks after a Multi-AZ failover of the replication instance. |
| | EBS storage full | Enables detailed debug logging for multiple log elements resulting in full storage due to AWS DMS logs. | Set up alerts when storage reaches 80% and expand the storage associated with the DMS replication instance. For more information, see [Why is my AWS DMS replication DB instance in a storage full state?](https://repost.aws/knowledge-center/dms-replication-instance-storage-full) |
| | Apply changes in the maintenance window | Modify the configuration for your DMS replication instance that results in downtime and select the “Apply in the next scheduled maintenance window” option. | DMS automatically resumes tasks after maintenance. |
| | Resource contention on the replica instance (high CPU, memory contention, higher than baseline IOPS) | Create multiple DMS tasks with high values ​​for MaxFullLoadSubTasks on a small DMS replica instance. | Set up monitoring and alerting on key CloudWatch metrics, as discussed in the Monitoring and Alerting section. Extend the instance class or you can move the tasks to a new replica instance. |
| | Upgrade the DMS replica instance | Upgrade the DMS backup instance. DMS no longer supports instancesThe DMS version is outdated, so users need to upgrade their backup instance. For more information, please refer to the [AWS DMS Release Notes.](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_ReleaseNotes.html) | To minimize downtime related to this activity, we recommend that you perform a thorough PoC. After testing the PoC, you can plan to create new replication instances running on the latest DMS version and move all your tasks to off-peak hours when the change data capture (CDC) latency is zero or minimal. For more information, see [Migrate a Task.](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Tasks.Moving.html) You can also see [Perform a side-by-side upgrade in AWS DMS by moving tasks to minimize business impact.](https://aws.amazon.com/blogs/database/perform-a-side-by-side-upgrade-in-aws-dms-by-moving-tasks-to-minimize-business-impact/) |
| Data issues | Data replication | Run a fully loaded task only twice, the first time stopping the task in the middle and the second time running the task with "DO NOTHING" configured for Target table prepare mode. | Use DMS validation for supported database engines. If validation reports any mismatches, you need to investigate based on the exact error. To mitigate the issue, you can perform a backfill by creating a full load or table reload (if applicable) task for the specific tables, and then create a continuous replication task. |
| | Data loss | Create triggers on the target to delete or truncate random records. | We recommend using DMS authentication to avoid these issues. You can perform a table reload or task, or create a new full load task and modify the data collection task to perform a new data load for the affected table(s). |
| | Table errors | Obtaining exclusive access locks on tables before starting the DMS task OR using unsupported data types. | This issue may be due to an unsupported feature or configuration of DMS. Investigation based on the exact error is required. For more information, please refer to [Why is my AWS DMS task in an error state?](https://repost.aws/knowledge-center/dms-task-error-status) |
| Latency issues | Swap file accumulation on the replicated instance | Initiate long-running transactions with large change counts and monitor the CloudWatch metric [CDCChangesDiskSource](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Monitoring.html). | Monitor the CDCChangesDiskSource and CDCChangesDiskTarget metrics. Refer to this Knowledge Center article for how to create an SOP: [What are swap files and why are they taking up space on my AWS DMS instance?](https://repost.aws/knowledge-center/dms-swap-files-consuming-space) |
| | BatchApply error | Delete a record on the target and update it on the source using BatchApply on the task. | Set up an alert on the DMS CloudWatch log for a failed bulk apply operation, please refer to the Monitoring and Alerting section for detailed instructions. For troubleshooting and creating SOPs, please refer to this Knowledge Center article: [How can I troubleshoot why Amazon Redshift switches to batch mode?](https://repost.aws/knowledge-center/dms-task-redshift-bulk-operation) |
| Data validation issues | Missing source | These can be simulated due to missing data on the source and target. | Review the supported use cases and limitations with [AWS DMS Data Validation](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Validating.html) and refer to the following Knowledge Center article for more information: [Why did my AWS DMS task validation fail or why is validation not progressing?](https://repost.aws/knowledge-center/dms-task-validation-failed-stuck) |
| | Missing Targets | These can be simulated due to missing data about the source and target. | Review the supported use cases and limitations with [AWS DMS Data Validation](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Validating.html) and refer to the following Knowledge Center article for more information: [Why did my AWS DMS task validation fail or why is validation not progressing?](https://repost.aws/knowledge-center/dms-task-validation-failed-stuck) |
| | Record Differences | You can create different table schemas in the source and target to simulate this scenario. | Review the supported use cases and limitations with [AWS DMS Data Validation](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Validating.html) and refer to the following Knowledge Center article for more information: [Why did my AWS DMS task validation fail or why is validation not progressing?](https://repost.aws/knowledge-center/dms-task-validation-failed-stuck) |
| | No tFound a qualified primary/unique key | Validation requires a primary or unique key in the table. LOBs and some other data types are not supported with DMS validation. For more details, see [validation restrictions.](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Validating.html#CHAP_Validating.Limitations) | Review the supported use cases and limitations with [AWS DMS Data Validation](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Validating.html) and refer to the following Knowledge Center article for more information: [Why did my AWS DMS task validation fail or why is validation not progressing?](https://repost.aws/knowledge-center/dms-task-validation-failed-stuck) |

By systematically testing these scenarios and documenting the results, organizations can develop robust recovery processes that address both common and unique failure modes. This proactive approach not only helps maintain system reliability but also provides operations teams with clear protocols for addressing issues as they arise.

## Develop Standard Operating Procedures (SOPs)

In your failure testing scenarios, carefully document the impact of each incident on your replication system. This documentation forms the basis for creating custom SOPs that your team can rely on when managing your AWS DMS deployment. The mitigation strategies outlined in our failure testing framework serve as a great starting point for developing these procedures.

Your initial SOPs will appear early in the PoC testing process. These procedures should be considered a living document, requiring regular updates and refinements as you gain operational experience and encounter new scenarios. The dynamic nature of database migrations means that your SOPs will evolve with your understanding of system behavior and emerging challenges.

For additional guidance on handling complex migration scenarios, we recommend reviewing our three-part blog series on debugging AWS DMS migrations. These resources provide valuable insights that can help you develop systematic approaches to troubleshooting, even for scenarios not covered in your existing SOPs. You can find these detailed instructions at:

1. [Debugging Your AWS DMS Migrations: What to Do When Things Go Wrong (Part 1)](https://aws.amazon.com/blogs/database/debugging-your-aws-dms-migrations-what-to-do-when-things-go-wrong-part-1/)
2. [Debugging Your AWS DMS Migrations: What to Do When Things Go Wrong (Part 2)](https://aws.amazon.com/blogs/database/debugging-your-aws-dms-migrations-what-to-do-when-things-go-wrong-part-2/)
3. [Debugging Your AWS DMS Migrations: What to Do When Things Go Wrong (Part 3)](https://aws.amazon.com/blogs/database/debugging-your-aws-dms-migrations-what-to-do-when-things-go-wrong-part-3/)

By documenting and testing these processes, organizations can accurately measure and validate the ability of their replication system to meet SLAs, especially in the event of a catastrophic failure. This proactive approach helps identify potential bottlenecks and areas for improvement in their disaster recovery strategy, thereby strengthening the resilience and reliability of their data replication architecture.

When designing a data replication strategy with AWS DMS, it is important to establish comprehensive contingency plans for scenarios involving service unavailability or data discrepancies. A thorough assessment of the organization’s RTO and RPO will drive the development of SOPs. This strategic planning not only promotes business continuity, but also provides valuable insights into the actual performance metrics of your replication system in failure scenarios.

## Monitoring and Alerting

To maximize the effectiveness of AWS DMS, a strategic approach to monitoring and reporting is required. A robust monitoring framework is essential to maintain seamless replication operations and promote data integrity throughout the migration process.

Configuring appropriate alerts during the initial setup process will provide real-time visibility into replication tasks and enable rapid response to anomalies. These monitoring capabilities act as an early warning system, helping to maintain the health and efficiency of your database migration infrastructure.

Implementing proactive monitoring and alerting improves operational reliability and provides insight into resource utilization and performance patterns. This systematic approach enables data-driven decision making and maintains optimal replication performance throughout the migration lifecycle.

AWS DMS providesprovides the following monitoring features:

1. [Amazon CloudWatch](https://aws.amazon.com/cloudwatch/) metrics – These metrics are automatically populated by AWS DMS for users to get insights into resource utilization and related metrics for individual tasks and at the replication instance level. For a list of all the metrics available with AWS DMS, refer to [AWS Database Migration Service Metrics.](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Monitoring.html#CHAP_Monitoring.Metrics)
2. AWS DMS CloudWatch logs and Time Travel logs – AWS DMS creates error logs and populates them based on the logging level set by the user for each component. For more information, see [View and manage AWS DMS task logs.](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Monitoring.html#CHAP_Monitoring.ManagingLogs) When CloudWatch logs are enabled, AWS DMS enables [context logging](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Monitoring.html#datarep_Monitoring_ContextLogging) by default. DMS also includes Time Travel logging to assist in debugging replication tasks. For more information about Time Travel logging, see [Setting Up Time Travel Tasks](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Tasks.CustomizingTasks.TaskSettings.TimeTravel.html). For best practices when using Time Travel logs, see [Troubleshooting Replication Tasks with Time Travel.](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_BestPractices.html#CHAP_BestPractices.TimeTravel)
3. Task and Table Status – AWS DMS provides near-real-time dashboards to report the status of tasks and tables. For a detailed list of task statuses, see [Task Status](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Monitoring.html#CHAP_Tasks.Status). For table status, see [Table Status During Task Execution.](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Monitoring.html#CHAP_Tasks.CustomizingTasks.TableState)
4. [AWS CloudTrail](https://aws.amazon.com/cloudtrail/) logs– AWS DMS is integrated with AWS CloudTrail, a service that provides a log of actions performed by users, roles, or AWS services in AWS DMS. CloudTrail records all AWS DMS API calls as events, including calls from the AWS DMS console and from code calls to AWS DMS API operations. For more information on setting up CloudTrail, see the [AWS CloudTrail User Guide](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/).

5. Monitoring dashboard – The enhanced monitoring dashboard provides comprehensive visibility into key metrics related to your monitoring tasks and replication instances; filtering, aggregating, and visualizing metrics for specific resources you want to monitor. The dashboard directly publishes existing CloudWatch metrics to monitor resource performance without changing the data point sampling time. For more information, please refer to [Overview of the enhanced monitoring dashboard](https://docs.aws.amazon.com/dms/latest/userguide/enhanced-monitoring-dashboard.html#overview-enhanced-monitoring-dashboard).

We recommend setting up CloudWatch alarms on key metrics and event logs to proactively identify potential issues before they escalate into system-wide disruptions. While this basic monitoring approach is just a starting point, it is important to expand your monitoring strategy based on your specific use case requirements and business goals.

| Metric Type | Metric Name | Remediation |
| :---: | :---: | :---: |
| Server Metrics | CPU Utilization | You should set up alerts based on these metrics to alert operators that resource contention is impacting the performance of your DMS tasks. Based on the resource limits on your server, you may need to upgrade your DMS instance class if there is CPU and memory contention, or increase storage if storage is low or baseline IOPS are limited. For more information on how to choose the right replication instance, you can refer to the article [Choosing the best size for a replication instance.](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_BestPractices.SizingReplicationInstance.html). |
| | Free Memory | | | | | SwapUsage | | | | | FreeStorageSpace | | | | | WriteIOPS | | | | ReadIOPS | | |
| Replication Task Metrics | CDCLatencySource | Based on SLA requirements, you can set alarm thresholds for latency metrics. In DMS, latency can be caused by many reasons. To troubleshoot and create SOPs, you can refer to [Troubleshooting Latency in AWS Database Migration Service. (AWS DMS)](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Troubleshooting_Latency.html) |
| | CDCLatencyTarget | |
| DMS Event for Replication Instance | ReplaceChange Configuration | Each of these categories is a category with different events associated with it. You can set up notifications for specific events based on your needs. For a detailed list and description of these events, please refer to the [AWS DMS Events and Event Notifications for SNS Notifications category.](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Events.html#USER_Events.Messages). |
| | Creation | |
| | Delete | |
| | Maintenance | |
| | Low Storage | | | | Failover | |
| | Error | |
| DMS Events for Replication Tasks | Error | Each of these categories is a category with different events associated with it. You can set up notifications for specific events based on your needs. For a detailed list and description of these events, please refer to the [AWS DMS events and event messages for SNS notifications.](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Events.html#USER_Events.Messages]category. |
| | Status Change | |
| | Creation | |
| | Delete | |

For a complete list of available metrics, you can refer to the AWS DMS User Guide for [AWS Database Migration Service](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Monitoring.html#CHAP_Monitoring.Metrics) metrics.

You can use [Amazon EventBridge](https://aws.amazon.com/eventbridge/) to provide notifications when AWS DMS events occur or use [Amazon Simple Notification Service](https://aws.amazon.com/sns/) (Amazon SNS) to create alerts for important events. For more information about EventBridge events in DMS, see [EventBridge Event User Guide.](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_EventBridge.html) For more information about using Amazon SNS with AWS DMS, see [Amazon SNS Event User Guide](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Events.html).

In addition to setting up CloudWatch alarms, you can create custom alarms based on AWS DMS CloudWatch error logs using metric filters. For step-by-step instructions on how to implement these custom alarms, see the blog post titled [\"Send Alerts on Custom AWS DMS Errors from Amazon CloudWatch Logs\"](https://aws.amazon.com/blogs/database/send-alerts-on-custom-aws-dms-errors-from-amazon-cloudwatch-logs/). This resource provides a comprehensive guide to enhancing your custom error monitoring capabilities.

## Apply the principles of the AWS Well-Architected Framework

The AWS Well-Architected Framework helps you understand the pros and cons of cloud-building decisions. The six pillars of the framework guide you through architectural best practices for designing and operating systems that are reliable, secure, efficient, cost-effective, and sustainable.

Using the [AWS Well-Architected Tool](https://aws.amazon.com/well-architected-tool/), available for free in the [AWS Management Console](https://console.aws.amazon.com/wellarchitected), you can review your workloads against these best practices by answering a set of questions for each pillar.

For more expert guidance and best practices for your cloud architecture—reference architecture implementations, diagrams, and white papers—see the [AWS Architecture Center](https://aws.amazon.com/architecture/)

## Conclusion

In this article, we presented a comprehensive framework for building resilient AWS DMS deployments. The effectiveness of this guide is directly related to the depth of your implementation and its adaptability to your specific environment. We strongly encourage organizations to carefully review each section of this guide and use it as a foundation to develop a customized migration strategy that fits your unique use case.

By carefully evaluating and incorporating these recommendations into your migration planning process, you can develop a comprehensive and reliable approach to using AWS DMS that will set you up for long-term success in your data migration strategies.

For additional support and resources, visit [AWS DMS documentation](https://docs.aws.amazon.com/dms/latest/userguide/Welcome.html) and engage with [AWS Support](https://aws.amazon.com/contact-us/).

## About the Author

**Sanyam Jain** is a Database Engineer on the AWS Database Migration Service (AWS DMS) team. He works closely with customers, providing technical guidance to migrate on-premises workloads to the AWS Cloud. He also plays a key role in enhancing the quality and functionality of AWS data migration products.

**Sushant Deshmukh** is a Senior Partner Solutions Architect, working with Global System Integrators. He is passionate about designing highly available, scalable, resilient, and secure architectures on AWS. He helps AWS Customers and Partners successfully migrate and modernize their applications to the AWS Cloud. In addition to hiswork, he enjoys traveling to explore new places and cuisines, playing volleyball, and spending quality time with family and friends.

**Alex Anto** is a Data Migration Specialist Solutions Architect on the Amazon Database Migration Accelerator team at Amazon Web Services. He works as an Amazon DMA Consultant, assisting AWS customers in migrating their on-premises data to AWS Cloud database solutions.