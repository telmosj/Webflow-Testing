# 4.5 Ensure CloudTrail configuration changes are monitored

## Summary
It's critical to keep a close eye on your AWS CloudTrail configuration to maintain visibility into the activities occurring in your AWS accounts. You should set up real-time monitoring to detect and alert on any changes made to your CloudTrail settings. This can be achieved by piping CloudTrail logs to CloudWatch Logs or an external SIEM solution and configuring metric filters and alarms.

## Where did this come from?
This recommendation comes from the CIS Amazon Web Services Foundations Benchmark v3.0.0 released on 01-31-2024. You can find more details and download a copy of the full benchmark from the [Center for Internet Security website](https://downloads.cisecurity.org/#/).

The CIS benchmarks provide consensus-based best practice recommendations for securely configuring IT systems and cloud services. You can find the official AWS documentation on monitoring CloudTrail events with CloudWatch Logs [here](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/monitor-cloudtrail-log-files-with-cloudwatch-logs.html).

## Who should care? 
This recommendation is relevant to:
- Cloud Security Engineers responsible for ensuring security visibility into AWS accounts
- DevOps Teams who manage the configuration of AWS monitoring and logging services 
- Compliance Officers that need to demonstrate appropriate security controls are in place
- IT Auditors assessing the configuration of AWS accounts against industry standards

## What is the risk?
Without monitoring enabled for CloudTrail configuration changes, you could lose visibility into activity in your AWS accounts if a malicious actor or accidental change disables or alters CloudTrail settings. 

This could allow suspicious behavior to go unnoticed, potentially leading to data breaches, interruptions to workloads, or abuse of resources for malicious purposes. Prompt detection of CloudTrail changes allows you to quickly investigate and respond.

## What's the care factor?
Monitoring CloudTrail configuration should be considered a high priority, foundational security control. Loss of visibility is a serious concern as it inhibits your ability to detect and investigate suspicious activity that could have material impacts on your business.

While the probability of a malicious insider or external threat actor explicitly targeting CloudTrail configuration may be relatively low in many organizations, the consequences have potential to be severe. Accidental misconfiguration or disabling of CloudTrail by administrators is also a risk that can be mitigated with appropriate monitoring.

## When is it relevant?
This control is relevant for any AWS account where CloudTrail is being used to log management events, which should be all accounts. The CIS Benchmark also recommends using CloudTrail to capture logs from all regions, even ones not actively used.

It is especially important for more sensitive accounts used for production workloads or that contain sensitive data. Less critical for test or development accounts, but still advisable.

## What are the trade offs?
Implementing this control requires some additional effort to configure the necessary metric filters and alarms, and ongoing costs for CloudWatch. However, these are relatively low compared to the security benefits.

The alerting requires an SNS topic with at least one subscriber, which will lead to some potential noise if the configuration is updated frequently for legitimate reasons. Filtering the alerts to only fire on high-risk events (like disabling logging) can help reduce unwanted noise.

The use of CloudWatch Logs also entails some cost, but overall implementing this control is relatively low friction. The main overhead is just the initial configuration effort.

## How to make it happen?

Here are step-by-step instructions to configure CloudWatch to monitor CloudTrail configuration changes:

1. Identify the CloudWatch Log Group used by your multi-region CloudTrail trail(s). You can find it by:

   a. Listing all CloudTrails with `aws cloudtrail describe-trails`
   
   b. Looking for trails with "IsMultiRegionTrail" set to true
   
   c. Note down the <cloudtrail_log_group_name> from the CloudWatchLogsLogGroupArn 
   
   d. Confirm the trail has logging active with `aws cloudtrail get-trail-status --name <trail_name>`

2. Create a CloudWatch Metric Filter on the CloudTrail Log Group to track key events:
```
aws logs put-metric-filter --log-group-name <cloudtrail_log_group_name> \
--filter-name `<cloudtrail_cfg_changes_metric>` \
--metric-transformations metricName=`<cloudtrail_cfg_changes_metric>`,metricNamespace='CISBenchmark',metricValue=1 \
--filter-pattern '{ ($.eventName = CreateTrail) || ($.eventName = UpdateTrail) || ($.eventName = DeleteTrail) || ($.eventName = StartLogging) || ($.eventName = StopLogging) }'
```

3. Create an SNS Topic to notify on alarm events:
```
aws sns create-topic --name <sns_topic_name>
```

4. Subscribe to the SNS Topic (can use email, lambda, etc):
```  
aws sns subscribe --topic-arn <sns_topic_arn> --protocol <protocol> --notification-endpoint <endpoint>
```

5. Create a CloudWatch Alarm on the Metric Filter that publishes to the SNS Topic:
```
aws cloudwatch put-metric-alarm --alarm-name `<cloudtrail_cfg_changes_alarm>` \
--metric-name `<cloudtrail_cfg_changes_metric>` \
--statistic Sum --period 300 --threshold 1 --comparison-operator GreaterThanOrEqualToThreshold \
--evaluation-periods 1 --namespace 'CISBenchmark' \
--alarm-actions <sns_topic_arn>  
```

## What are some gotchas?
To set up the CloudWatch Alarms, you'll need appropriate permissions to CloudTrail, CloudWatch Logs, CloudWatch Metrics, and SNS. 

Example IAM permissions required:
- cloudtrail:DescribeTrails
- cloudtrail:GetTrailStatus
- cloudtrail:GetEventSelectors  
- logs:DescribeMetricFilters
- logs:PutMetricFilter
- cloudwatch:DescribeAlarms
- cloudwatch:PutMetricAlarm
- sns:CreateTopic
- sns:Subscribe 
- sns:ListSubscriptionsByTopic

Make sure you have the AWS CLI installed and properly configured with credentials that have the necessary permissions to run the commands and create the required resources.

It's also important that the CloudTrail trail being monitored is properly configured to capture all management events from all regions. You can confirm this by checking the EventSelectors.

## What are the alternatives?
Instead of using the native AWS CloudTrail and CloudWatch integration, you could stream CloudTrail logs to an external SIEM or log analytics solution. Most 3rd party platforms like Splunk, SumoLogic, ELK stack, Azure Sentinel etc have builtin support for ingesting CloudTrail logs and can provide similar abilities to alert on configuration changes.

Some organizations may also use infrastructure-as-code and version controlled configurations for CloudTrail. In those cases, unauthorized configuration drift can be detected through the CI/CD pipeline or other config management tools without relying on event-based monitoring through CloudWatch.

## Explore further
- For more details on working with CloudTrail event logs in CloudWatch, see the [AWS documentation](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/view-cloudtrail-events-cloudwatch-logs.html).
- Another useful resource is the [AWS CloudTrail Security Best Practices](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/best-practices-security.html) guide.
- The CIS AWS Benchmark contains related recommendations like ensuring CloudTrail is enabled in all regions (section 4.1), logs are encrypted (4.2), access is controlled (4.3) and that an SNS topic exists for notification on key events (4.4).
- More broadly, this control supports the detective capabilities described in the CIS Top 20 Controls, specifically [Control 8: Audit Log Management](https://www.cisecurity.org/controls/cis-controls-list/).