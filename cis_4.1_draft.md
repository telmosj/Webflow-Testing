# 4.1 Ensure unauthorized API calls are monitored (Manual)

## Summary
It's critical to keep a watchful eye on your AWS environment for any suspicious activity. One key way to do this is to monitor for unauthorized API calls in real-time. By setting up the right monitoring and alerts, you can quickly detect potential security incidents and spring into action to investigate and remediate.

## Where did this come from?
This recommendation comes from the CIS Amazon Web Services Foundations Benchmark v3.0.0 - 01-31-2024. You can download the full benchmark document from the [CIS website](https://downloads.cisecurity.org/#/). 

The benchmark provides prescriptive guidance for configuring security options for a subset of Amazon Web Services with an emphasis on foundational, testable, and architecture agnostic settings. Recommendation 4.1 falls under the "Monitoring" category.

For more background, check out the [AWS CloudTrail documentation](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-user-guide.html) and the [Amazon CloudWatch Logs documentation](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/WhatIsCloudWatchLogs.html).

## Who should care?
This recommendation is super relevant for:
- Cloud Security Engineers with responsibility for detecting and responding to threats 
- DevOps Engineers with ownership over the AWS environment configuration
- Cloud Architects with a mandate to design secure and compliant systems
- Security Managers with accountability for the overall security posture

## What is the risk?
The main risk we're looking at here is an attacker making unauthorized API calls to perform malicious actions in your AWS environment. This could include things like:
- Launching new EC2 instances for crypto mining 
- Creating IAM users and granting them excessive permissions
- Deleting resources like RDS databases or S3 buckets
- Exfiltrating sensitive data to an external location

Without proper monitoring of unauthorized API calls, malicious activity could go undetected for an extended period, giving attackers a dangerous foothold. The sooner you can detect and respond, the better you can contain the blast radius.

## What's the care factor? 
Monitoring for unauthorized API calls should be a high priority for any organization running workloads on AWS. It's a foundational security control that acts as an early warning system. Attacks often start with a few unauthorized API calls that then escalate into major incidents.

So while it may not stop attacks altogether, it puts you in a much better position to identify malicious activity early in the kill chain before serious damage is done. Think of it like a smoke alarm - catching a fire early is way better than realizing your house has already burnt down!

## When is it relevant?
This control should be implemented in basically any AWS environment, with a few potential exceptions:
- Static website hosting (e.g. S3 static website) with no other AWS services in use
- Standalone containerized workloads with no access to the AWS control plane
- Completely isolated development/test environments with no sensitive data

Outside of these limited cases, monitoring unauthorized API calls is almost always a smart move. It's especially critical for:
- Environments hosting sensitive data (e.g. PII, PHI, financial records) 
- Regulated workloads subject to compliance standards (e.g. PCI DSS, HIPAA, SOC 2)
- Mission-critical production systems 
- Complex environments with many AWS services and API calls

## What are the trade offs?
The main cost of implementing this control is the effort required to configure the CloudWatch Logs metric filters and alarms. It's not a huge lift, but does require some thoughtful setup to avoid too many false positives. 

There's also the operational overhead of responding to alerts generated. Someone needs to be responsible for triaging the notifications, investigating the activity, and determining the appropriate response. Too many noisy alerts can lead to alert fatigue.

From a performance or user experience perspective, there really isn't much downside. Monitoring doesn't add any latency to the API calls themselves. And these are generally calls initiated by applications or scripts, not interactive users.

## How to make it happen?
Alright, let's dive into the nitty gritty. We're going to assume you already have CloudTrail enabled to log API activity to CloudWatch Logs. If not, start there first! Then come back.

Here's the step-by-step:

1. Create a new CloudWatch Logs metric filter 

```
aws logs put-metric-filter \
--log-group-name <cloudtrail_log_group_name> \
--filter-name "UnauthorizedAPICalls" \  
--metric-transformations \
metricName=UnauthorizedAPICalls,metricNamespace=CISBenchmark,metricValue=1 \
--filter-pattern '{ ($.errorCode ="*UnauthorizedOperation") || ($.errorCode ="AccessDenied*") && ($.sourceIPAddress!="delivery.logs.amazonaws.com") && ($.eventName!="HeadBucket") }'
```

This metric filter will parse your CloudTrail logs and create a custom metric in the CISBenchmark namespace any time an API call with an "UnauthorizedOperation" or "AccessDenied" error code is logged. 

2. Create an SNS topic to send alerts to

```
aws sns create-topic --name CISUnauthorizedAPICallAlerts
```

Note down the TopicArn that is returned. You'll need it in the next step.

3. Create a CloudWatch alarm for the unauthorized API calls metric

```
aws cloudwatch put-metric-alarm \
--alarm-name CISUnauthorizedAPICallsAlarm \
--metric-name UnauthorizedAPICalls \
--statistic Sum \
--period 300 \
--threshold 1 \
--comparison-operator GreaterThanOrEqualToThreshold \
--evaluation-periods 1 \
--namespace CISBenchmark \
--alarm-actions <sns_topic_arn>
```

This alarm is configured to trigger and send a notification to the SNS topic anytime there is at least 1 unauthorized API call within a 5 minute period. Adjust the period and threshold as needed for your environment.

4. Subscribe your team's email addresses to the SNS topic
Use the AWS console or CLI to subscribe the appropriate email addresses to receive the alerts. You can also integrate with incident management or chat tools like PagerDuty or Slack.

## What are some gotchas?
A few things to keep in mind when implementing this control:

- Make sure you exclude "HeadBucket" events from the metric filter, as these will often fail with an "AccessDenied" error even for normal behavior when a user/role doesn't have S3 ListBucket permissions
- Similarly, the "delivery.logs.amazonaws.com" source IP address should be excluded since this is CloudTrail itself delivering the log files to S3. Failed delivery attempts will show up as "UnauthorizedOperation".
- Your team will need access to CloudWatch Logs, CloudWatch, and SNS to set this up. The specific permissions required are:
  - [logs:PutMetricFilter](https://docs.aws.amazon.com/AmazonCloudWatchLogs/latest/APIReference/API_PutMetricFilter.html) 
  - [logs:DescribeMetricFilters](https://docs.aws.amazon.com/AmazonCloudWatchLogs/latest/APIReference/API_DescribeMetricFilters.html)
  - [cloudwatch:PutMetricAlarm](https://docs.aws.amazon.com/AmazonCloudWatch/latest/APIReference/API_PutMetricAlarm.html)
  - [cloudwatch:DescribeAlarms](https://docs.aws.amazon.com/AmazonCloudWatch/latest/APIReference/API_DescribeAlarms.html)  
  - [sns:CreateTopic](https://docs.aws.amazon.com/sns/latest/api/API_CreateTopic.html)
  - [sns:Subscribe](https://docs.aws.amazon.com/sns/latest/api/API_Subscribe.html)
  - [sns:ListSubscriptionsByTopic](https://docs.aws.amazon.com/sns/latest/api/API_ListSubscriptionsByTopic.html)
  
## What are the alternatives?
If you're already using a SIEM like Splunk, SumoLogic, or ELK, you can send your CloudTrail logs there instead of CloudWatch Logs. Most SIEMs have rule templates for detecting unauthorized API activity that you can enable. 

Some folks use open source tools like [these alternatives to Splunk, SumoLogic](https://alternativeto.net/software/splunk/?license=opensource).