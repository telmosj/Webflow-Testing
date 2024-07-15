# 4.7 Ensure disabling or scheduled deletion of customer created CMKs is monitored (Manual)

## Summary
It's important to keep a close eye on your customer managed Customer Master Keys (CMKs) in AWS Key Management Service (KMS). If someone disables or schedules deletion of a CMK, any data encrypted with that key will become inaccessible. Setting up monitoring and alarms for these events can help you stay on top of unexpected changes to your CMKs.

## Where did this come from?
This recommendation comes from the CIS Amazon Web Services Foundations Benchmark v3.0.0 - 01-31-2024. You can download the full benchmark from the [Center for Internet Security website](https://downloads.cisecurity.org/#/). The benchmark provides prescriptive guidance for configuring security options for a subset of Amazon Web Services with an emphasis on foundational, testable, and architecture agnostic settings. 

For more details on working with CMKs in KMS, check out the [AWS KMS Developer Guide](https://docs.aws.amazon.com/kms/latest/developerguide/overview.html).

## Who should care? 
- Security engineers with responsibility for ensuring data is not inadvertently made inaccessible
- Compliance officers with oversight of encryption key management practices
- Developers and operations staff with ability to manage CMKs

## What is the risk?
The main risk is losing access to important data if a CMK used to encrypt that data is disabled or deleted. Without the CMK, there is no way to decrypt the data, rendering it unusable. 

Monitoring CMK state changes can help detect either malicious activity (e.g. an attacker trying to make data unavailable) or accidental misconfiguration (e.g. an admin disabling the wrong key). Quick detection enables prompt response to investigate and remediate the situation.

## What's the care factor?
For data encrypted with customer managed CMKs, the care factor for this recommendation should be high. Loss of data availability can significantly impact business operations. The reputational and financial costs of a major data loss event are substantial.

However, the care factor is lower for data encrypted with AWS managed CMKs, since you don't have the ability to disable or delete those keys yourself. It's also lower for non-critical data that doesn't require long-term availability.

## When is it relevant?
This recommendation is most relevant when:
- You are using customer managed CMKs in KMS to encrypt sensitive data
- That sensitive data must remain accessible for a substantial length of time (months or years)
- Multiple people or processes have the ability to manage the lifecycle of CMKs

It is less relevant for use cases where:
- Only AWS managed CMKs are used
- Encrypted data has a short lifespan and is not needed long-term
- CMKs are managed by a small operations team following a rigorous process

## What are the trade offs?
The main cost of this recommendation is the effort required to configure the CloudWatch alarms and respond to notifications. There may be some minor cost for the CloudTrail logs and CloudWatch metrics/alarms but it's usually negligible.  

Following this recommendation should have no performance impact and minimal effect on the end user experience when accessing encrypted data. The security benefit of increased visibility into the status of CMKs usually outweighs the minimal overhead.

## How to make it happen?
The high-level steps to implement this recommendation are:

1. Ensure you have an active CloudTrail trail capturing management events across all regions. 
2. Create a new CloudWatch Logs metric filter on the CloudTrail log group to match KMS `DisableKey` and `ScheduleKeyDeletion` events.
3. Create a CloudWatch alarm for when the KMS state change metric is greater than zero over a 5 minute period.
4. Configure the alarm to send notifications to an SNS topic.
5. Subscribe your team to the SNS topic to receive the alerts.

Here are the detailed AWS CLI commands:

Find the CloudTrail log group name:
```
aws cloudtrail describe-trails \
  --query 'trailList[?IsMultiRegionTrail==`true`].CloudWatchLogsLogGroupArn'
```

Create CloudWatch metric filter on that log group:
```
aws logs put-metric-filter \
  --log-group-name <cloudtrail_log_group_name> \
  --filter-name KMSCustomerKeyDisableOrDeleteEvents \  
  --metric-transformations \
      metricName=KMSCustomerKeyDisableOrDeleteEvents,\
      metricNamespace='CISBenchmark',\
      metricValue=1 \
  --filter-pattern '{($.eventSource = kms.amazonaws.com) &&
                     (($.eventName=DisableKey) || 
                      ($.eventName=ScheduleKeyDeletion))
                    }'
```

Create SNS topic: 
```
aws sns create-topic --name KMSCustomerKeyEvents
```

Create CloudWatch alarm:
```  
aws cloudwatch put-metric-alarm \
  --alarm-name KMSCustomerKeyDisableOrDeleteAlarm \
  --metric-name KMSCustomerKeyDisableOrDeleteEvents \
  --statistic Sum \
  --period 300 \
  --threshold 1 \
  --comparison-operator GreaterThanOrEqualToThreshold \
  --evaluation-periods 1 \
  --namespace 'CISBenchmark' \
  --alarm-actions <sns_topic_arn>
```

Subscribe your team's email to the SNS topic:
```
aws sns subscribe \
  --topic-arn <sns_topic_arn> \
  --protocol email \
  --notification-endpoint my-email@example.com
```

## What are some gotchas?
A few things to watch out for when implementing this recommendation:

- Make sure the IAM role or user configuring these alarms has the necessary permissions. Key ones include `cloudtrail:DescribeTrails`, `logs:PutMetricFilter`, `logs:DescribeMetricFilters`, `cloudwatch:PutMetricAlarm`, and `sns:CreateTopic`. See the [AWS CloudTrail](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-permissions-for-sns-notifications.html) and [Amazon CloudWatch](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/iam-identity-based-access-control-cw.html) docs for more details.

- If you already have an SNS topic for receiving CloudTrail notifications, you can re-use that existing topic instead of creating a new one each time. 

- Note that the alarm metrics and notifications can incur minor costs. See the [AWS CloudTrail pricing](https://aws.amazon.com/cloudtrail/pricing/) page for details.

- Keep in mind these alarms only detect and notify you of CMK state changes. You'll still need to follow up and investigate the cause and impact of the event.

## What are the alternatives?
Other options for monitoring changes to your KMS CMKs include:

- Sending CloudTrail events to a third-party SIEM or log analysis platform that can alert on KMS events of interest. See this [blog post](https://aws.amazon.com/blogs/mt/how-to-integrate-aws-cloudtrail-with-splunk/) for an example of integrating CloudTrail with Splunk.

- Using [AWS Config](https://aws.amazon.com/config/) to track configuration changes to your KMS keys over time and compare against desired settings. See the [Monitoring AWS KMS with AWS Config](https://docs.aws.amazon.com/kms/latest/developerguide/monitoring-key-configuration-with-config.html) docs for guidance.

- Preventing unintended CMK deletions altogether by applying a [key policy](https://docs.aws.amazon.com/kms/latest/developerguide/key-policies.html) that denies the `kms:ScheduleKeyDeletion` and `kms:DisableKey` actions. Be cautious with this approach, as it may break legitimate key management workflows.

## Explore further

- CIS AWS Foundations Benchmark Recommendation 4.5 - Ensure a log metric filter and alarm exist for CloudTrail configuration changes
- CIS AWS Foundations Benchmark Recommendation 4.6 - Ensure a log metric filter and alarm exist for AWS Management Console authentication failures
- AWS User Guide - [Creating an Amazon CloudWatch Alarm to Detect Changes to Your IAM Configuration](https://docs.aws.amazon.com/IAM/latest/