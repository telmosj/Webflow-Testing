# 4.9 Ensure AWS Config configuration changes are monitored (Manual)

## Summary
It's always a good idea to keep an eye on what's going on with your AWS Config setup. You want to make sure you're notified if anyone makes changes that could impact your visibility into the state of your AWS environment. Setting up real-time monitoring for AWS Config configuration changes is a smart move to stay on top of things.

## Where did this come from?
This recommendation comes straight from the CIS Amazon Web Services Foundations Benchmark v3.0.0 - 01-31-2024. You can download the full benchmark document from the [CIS website](https://downloads.cisecurity.org/#/). It outlines a bunch of security best practices for configuring your AWS environment. For more nitty gritty details, check out the [AWS Config documentation](https://docs.aws.amazon.com/config/latest/developerguide/WhatIsConfig.html).

## Who should care? 
* Cloud Security Engineers responsible for monitoring the AWS environment
* DevOps Engineers who manage the AWS infrastructure and configurations
* Compliance Officers tasked with ensuring adherence to security benchmarks

## What is the risk?
If changes are made to your AWS Config setup without you knowing about it, you could lose visibility into important configuration items in your AWS account. This means potential misconfigurations or security gaps could fly under the radar. The sooner you're alerted to AWS Config changes, the faster you can verify if they were legit or not. This CIS recommendation helps you quickly detect and respond to unexpected modifications.

## What's the care factor?
On a scale of "meh" to "drop everything and fix it now", this one ranks pretty high up there. Visibility is key for staying secure in the cloud. You can't protect what you can't see. If your AWS Config settings get changed or disabled, you're flying blind. Considering AWS Config is your source of truth for configuration monitoring, you definitely want to care about unintended changes. Make this a priority to implement.

## When is it relevant?
This recommendation is relevant in basically any AWS environment where you care about security and compliance (so...all of them?). The only potential exception is if you have a super simple setup with like one EC2 instance and nothing else. But even then, why not monitor it anyway? There's really no downside. As your usage of AWS services grows, this becomes even more critical.

## What are the trade offs?
The main "cost" of this CIS recommendation is the time and effort to set it up initially. You'll need to configure CloudWatch and potentially an external SIEM to ingest the AWS Config logs and alert on changes. There's also the minor cost of storing the logs and running the metric alarms, but it's pretty negligible in the grand scheme of things. The ongoing overhead is minimal once it's all set up. The benefits of having this monitoring in place far outweigh the costs.

## How to make it happen?
Alright, let's get down to brass tacks. Here's how you can implement this:

1. Make sure you have a CloudTrail trail set up to capture AWS Config events. Enable the trail and double-check that it's logging.

2. Create a new CloudWatch Logs metric filter to look for the specific AWS Config API calls that modify the configuration:
```
aws logs put-metric-filter --log-group-name <your_cloudtrail_log_group> --filter-name "ConfigChanges" --metric-transformations metricName="ConfigChangesCount",metricNamespace="CISBenchmark",metricValue=1 --filter-pattern '{ ($.eventSource = config.amazonaws.com) && (($.eventName=StopConfigurationRecorder)||($.eventName=DeleteDeliveryChannel) ||($.eventName=PutDeliveryChannel)||($.eventName=PutConfigurationRecorder)) }'
```

3. Create an SNS topic to notify on alarm conditions:
```  
aws sns create-topic --name ConfigChangesNotify
```

4. Subscribe your email address (or some other endpoint) to the SNS topic:
```
aws sns subscribe --topic-arn <sns_topic_arn> --protocol email --notification-endpoint youremail@company.com  
```

5. Finally, create the CloudWatch alarm to alert when the metric filter condition is breached:
```
aws cloudwatch put-metric-alarm --alarm-name "ConfigChangesAlarm" --metric-name "ConfigChangesCount" --statistic Sum --period 300 --threshold 1 --comparison-operator GreaterThanOrEqualToThreshold --evaluation-periods 1 --namespace "CISBenchmark" --alarm-actions <sns_topic_arn>
```

That's the core of it! Obviously integrate this with your preferred alerting and incident response workflows as needed. You can customize the SNS subscriptions to notify your on-call rotation, ticketing system, Slack channel, or whatever you use.

## What are some gotchas?
There are a few things to watch out for when setting this up:

* Make sure your IAM user/role has the necessary permissions for CloudTrail, CloudWatch Logs, and SNS. The key ones are `cloudtrail:DescribeTrails`, `cloudtrail:GetTrailStatus`, `cloudtrail:GetEventSelectors`, `logs:DescribeMetricFilters`, `logs:PutMetricFilter`, `cloudwatch:DescribeAlarms`, `cloudwatch:PutMetricAlarm`, `sns:CreateTopic`, `sns:Subscribe`, and `sns:ListSubscriptionsByTopic`.  Check the [AWS IAM docs](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_actions-resources-contextkeys.html) for the full list of permissions for each service.

* Ensure you have a multi-region CloudTrail trail enabled to capture activity across all regions. The AWS Config APIs are considered global service events.

* Keep in mind that the alarm will trigger on ANY matched AWS Config change event. Be prepared to handle some false positives for intended configuration changes. 

* The provided filter pattern is quite broad and may result in extra noise. Feel free to tweak it to be more specific if you know exactly which API calls to monitor.

## What are the alternatives?
Another way to stay on top of AWS Config changes is to use the built-in AWS Config rules feature. You can set up a [config rule](https://docs.aws.amazon.com/config/latest/developerguide/evaluate-config_manage-rules.html) that is triggered by configuration changes and sends a notification. This is more tightly integrated with AWS Config itself.

Alternatively, you could use a 3rd party tool or service that specializes in AWS monitoring and alerting. Some options in this space include CloudCheckr, Dome9, and Palo Alto Networks Prisma Cloud. These tools often provide more user-friendly interfaces and out-of-the-box compliance rule sets.

## Explore further 
* Walk through a hands-on tutorial on [setting up AWS Config monitoring with CloudTrail and CloudWatch Events](https://docs.aws.amazon.com/config/latest/developerguide/monitor-config-with-cloudwatchevents.html) 
* Learn more about [AWS CloudTrail event log contents](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-event-reference-record-contents.html) to understand the structure of the data you're monitoring
* Check out the [CIS AWS Foundations Benchmark overview](https://d0.awsstatic.com/whitepapers/compliance/AWS_CIS_Foundations_Benchmark.pdf) whitepaper for more context on the other related recommendations
* Dive into the relevant CIS controls for this recommendation:
  * 8.11 Conduct Audit Log Reviews
  * 11.2 Document Traffic Configuration Rules
  * 16.1 Maintain an Inventory of Authentication Systems

And there you have it! A rundown of why this CIS recommendation matters, how to put it into practice, and some helpful resources to learn even more. Now go forth and monitor those AWS Config changes like a boss.