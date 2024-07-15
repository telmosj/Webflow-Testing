# 3.3 Ensure AWS Config is enabled in all regions (Automated)

## Summary

AWS Config is a powerful service that every AWS customer should enable across all regions in their account. It acts as a configuration management tool, keeping a detailed inventory of all supported resources. This includes snapshots of configurations, relationships between resources, and a full history of changes.

## Where did this come from?

This recommendation comes from the CIS Amazon Web Services Foundations Benchmark v3.0.0 - 01-31-2024. You can download the full benchmark document from the [CIS website](https://downloads.cisecurity.org/#/). The benchmark provides prescriptive guidance for configuring AWS accounts with security best practices in mind. You can learn more about AWS Config in the [AWS documentation](https://docs.aws.amazon.com/config/latest/developerguide/WhatIsConfig.html).

## Who should care? 

* Security engineers with responsibility for ensuring the security posture of AWS accounts
* Cloud architects with a need to understand resource configurations and relationships
* Compliance officers with audit requirements around configuration change management
* Developers with a desire for visibility into infrastructure-as-code deployments

## What is the risk?  

Without AWS Config enabled, organizations lack visibility into their resource configurations across regions. This introduces several risks:

* Resources could be misconfigured, leading to security vulnerabilities 
* Unintended changes could be made to production resources, causing outages
* Auditors may be unable to verify compliance with internal policies or external regulations
* Incident responders may struggle to determine root cause without configuration history

Enabling AWS Config across all regions significantly mitigates these risks by providing a centralized inventory, configuration history, and change notifications. While it doesn't prevent misconfigurations or unauthorized changes, it allows organizations to quickly detect and respond to them.

## What's the care factor?

For most organizations, enabling AWS Config should be considered a high priority, baseline security control. The information it provides is invaluable for security and compliance. It requires very little effort to set up, and once enabled, runs automatically in the background. The only potential downside is the storage costs for the configuration snapshots and logs, but this is minimal for all but the very largest AWS environments.

## When is it relevant?

This recommendation is relevant for virtually every AWS account. The only situations where it may not apply are:

* Temporary accounts used for short-term testing or demonstrations
* Standalone accounts with a small number of static resources
* Highly sensitive government or regulated environments with specific monitoring tools

Even in these edge cases, the benefits of AWS Config likely still outweigh the costs. It's a widely applicable best practice.

## What are the trade offs?

Enabling AWS Config does come with some minor costs and considerations:

* Storage costs for configuration snapshots and change logs, which vary based on number of resources
* Slight increase in IAM complexity to provision the necessary permissions 
* Potential duplication with other configuration management or CMDB tools

However, for the vast majority of organizations, these trade offs are minimal compared to the substantial security and compliance benefits.

## How to make it happen?

Enabling AWS Config is straightforward and can be done from the Management Console or CLI. Here's the process:

1. Navigate to the AWS Config console in the desired region
2. If Config is not already enabled, click "Get Started"
3. On the Settings page, select "Record all resources supported in this region"
4. Check "Include global resources" if this is the first or only region (for capturing IAM)
5. Choose an existing S3 bucket or create a new one for storing configuration snapshots
6. Choose an existing SNS topic or create one for change notifications
7. Click through to confirm and enable Config
8. Repeat this process for each region

It's worth noting that you'll need appropriate IAM permissions for setting up Config, specifically access to the Config, S3, and SNS services. The specific permissions are `config:Put*` for Config, `s3:GetBucketPolicy` and `s3:PutBucketPolicy` for S3, and `sns:GetTopicAttributes` and `sns:SetTopicAttributes` for SNS.

Alternatively, you can enable Config via the AWS CLI with a few commands:

1. Create an IAM role for Config with the necessary S3 and SNS permissions 
2. Run `aws configservice put-configuration-recorder` to enable resource recording
3. Run `aws configservice put-delivery-channel` to set up the S3 bucket and SNS topic
4. Run `aws configservice start-configuration-recorder` to start Config

Detailed steps and example CLI commands can be found in the [AWS Config Developer Guide](https://docs.aws.amazon.com/config/latest/developerguide/gs-cli-subscribe.html).

## What are some gotchas?

There are a few things to watch out for when enabling AWS Config:

* Config needs permission to read and write to the S3 bucket and SNS topic. Make sure the IAM role has the necessary permissions (`s3:PutObject`, `s3:GetBucketAcl`, `sns:Publish`)
* The S3 bucket and SNS topic need to be in the same region as the Config recorder
* If you have a multi-account environment with AWS Organizations, you can set up Config as a delegated administrator for central management, but this requires additional setup
* Some resources, like Lambda functions and DynamoDB tables, need additional permissions to fully record all attributes (`lambda:GetFunction`, `dynamodb:DescribeTable`)

More details on these gotchas and how to resolve them can be found in the [AWS Config FAQ](https://aws.amazon.com/config/faq/).

## What are the alternatives?

While AWS Config is the recommended native solution, there are some alternative options for configuration management and change tracking:

* Open source tools like [CloudCustodian](https://cloudcustodian.io/) or [Prowler](https://github.com/prowler-cloud/prowler) that use the AWS APIs to inventory resources
* Third-party SaaS solutions like [CloudCheckr](https://cloudcheckr.com/) or [Sonrai](https://sonraisecurity.com/) that provide additional compliance and security features
* Configuration management databases (CMDBs) that integrate with AWS, like [ServiceNow](https://www.servicenow.com/products/it-operations-management.html) or [Device42](https://www.device42.com/)

These may be appropriate for organizations with specific needs around compliance reporting or integration with existing tooling. However, for most, AWS Config is the simplest and most cost-effective solution.

## Explore further

* AWS Config [documentation](https://docs.aws.amazon.com/config/latest/developerguide/WhatIsConfig.html) 
* AWS Config [FAQ](https://aws.amazon.com/config/faq/)
* AWS Config [workshop](https://mng.workshop.aws/config.html)
* CIS Benchmarks [website](https://www.cisecurity.org/cis-benchmarks/)
* Related CIS AWS Controls: 1.4 Maintain Detailed Asset Inventory, 11.2 Document Traffic Configuration Rules, 16.1 Maintain an Inventory of Authentication Systems

I hope this article provides a helpful overview of the importance of enabling AWS Config across regions, with actionable steps and considerations for implementation. Let me know if you have any other questions!