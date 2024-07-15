# 4.13 Ensure route table changes are monitored (Manual)

Hey there! Let's chat about an important AWS security recommendation from the fine folks at the Center for Internet Security (CIS). They've got some great ideas for keeping your cloud environment safe and sound.

## Summary

CIS recommends monitoring changes to your AWS route tables in real-time. This helps ensure network traffic in your Virtual Private Cloud (VPC) flows as expected and prevents unauthorized modifications. You can do this by sending CloudTrail logs to CloudWatch or a SIEM and setting up metric filters and alarms.

## Where did this come from?

This recommendation comes straight from the "CIS Amazon Web Services Foundations Benchmark v3.0.0 - 01-31-2024". You can download the full benchmark from the [CIS website](https://downloads.cisecurity.org/#/). The benchmark provides a comprehensive set of security guidelines for configuring AWS. For more context, check out the [AWS CloudTrail documentation](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-user-guide.html) and the [AWS CloudWatch documentation](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/WhatIsCloudWatch.html).

## Who should care? 

This matters for:
- Cloud engineers responsible for designing and maintaining secure AWS networks 
- DevOps teams that deploy applications and services into VPCs
- Security analysts who monitor AWS environments for potential threats

## What is the risk?

Without monitoring route table changes, you could miss:
- Accidental misconfigurations that send traffic to the wrong places
- Malicious attempts to redirect traffic for eavesdropping or disruption
- Configuration drift that violates security policies over time

While major incidents are unlikely, proactively detecting unauthorized changes reduces risk exposure.

## What's the care factor?

On a scale from "meh" to "mega important", this falls somewhere in the middle. It's a defence-in-depth measure that helps catch issues early. Not end-of-the-world critical, but certainly worthwhile for security-conscious organizations. The effort to implement is fairly low, so there's little reason not to do it.

## When is it relevant?

This recommendation applies any time you're running workloads in a VPC. That's pretty much always these days. However, it's most important for:
- VPCs with multiple subnets and complex networking 
- Deployments with strict security and compliance requirements
- Dynamic environments where network configurations change frequently 

For a basic single-subnet VPC, it's okay to skip. But in general, more monitoring is better.

## What are the trade-offs?

The main costs are:
- Extra log storage in CloudWatch or your SIEM
- Potential noise from frequent alarms if route tables change often 
- Some upfront engineering effort to configure the monitoring

But these are minor compared to the security benefits. Performance and functionality of your VPC are unaffected. This is an easy win.

## How to make it happen?

The basic steps are:

1. Make sure you have a multi-region CloudTrail configured to capture management events. 
2. Create a new CloudWatch Logs metric filter for the CloudTrail log group.
3. Use a filter pattern that watches for specific API calls such as CreateRoute,CreateRouteTable,DeleteRoute etc
4. Create a CloudWatch alarm that triggers when the metric filter condition is breached.
5. Have the alarm send a notification to an SNS topic.
6. Subscribe your email, phone number, ticketing system, etc to the SNS topic.

The detailed CLI commands to do all this are in the CIS benchmark. It's a straightforward copy-paste job.

## What are some gotchas?

A few things to watch out for:
- The CloudTrail trail must be set to log management events. Data events alone aren't enough.
- The IAM user or role that creates the metric filter and alarm needs the appropriate permissions, like logs:PutMetricFilter and cloudwatch:PutMetricAlarm.  
- If you're using CloudFormation or Terraform to manage your VPC config, the monitoring may need to account for expected changes to avoid false alarms.
- SNS subscriptions may need to be confirmed before notifications start flowing.

## What are the alternatives?

Instead of using CloudWatch Logs and metric filters, you could:
- Stream CloudTrail events to a 3rd party SIEM for fancier rule logic
- Use the AWS Config service to track configuration changes
- Write custom code to analyze CloudTrail logs in S3 and alert on changes
- Lock down your route tables and monitor IAM permissions instead

But for most, the CIS recommendation is the simplest path forward.

## Explore further

Want to go deeper? Check out:
- CIS AWS Foundations Benchmark Recommendation 3.x for more CloudTrail best practices
- [CIS Control 8](https://www.cisecurity.org/controls/cis-controls-navigator/v8): Audit Log Management
- [CIS Control 12](https://www.cisecurity.org/controls/cis-controls-navigator/v8): Network Infrastructure Management
- [The AWS Security Logging & Monitoring Whitepaper for more logging ideas](https://docs.aws.amazon.com/whitepapers/latest/introduction-aws-security/monitoring-and-logging.html)

There you have it! A quick primer on monitoring those route tables. Stay safe out there and happy cloud computing!