# 4.6 Ensure AWS Management Console authentication failures are monitored (Manual)

Have you ever wondered how secure your AWS account really is? What if a malicious hacker was trying to break in right now by guessing your password over and over? Would you even know it was happening before it was too late? Never fear, intrepid AWS administrator, there's a solution to give you some peace of mind - monitoring AWS Management Console authentication failures!

## Where did this come from?

This sage advice comes straight from the super smart folks at the Center for Internet Security (CIS). Specifically, it's recommendation 4.6 in the CIS Amazon Web Services Foundations Benchmark v3.0.0 released on 01-31-2024. You can download the full benchmark with all the juicy technical details directly from the CIS website: https://downloads.cisecurity.org/#/

For more background, check out these helpful docs from AWS on using CloudTrail to monitor activity in your account:
- https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-user-guide.html
- https://docs.aws.amazon.com/awscloudtrail/latest/userguide/receive-cloudtrail-log-files-from-multiple-regions.html  

## Who should care?

This matters for:
- Cloud Security Engineers looking to level up the detection capabilities in their AWS environments
- IT Managers and Directors who want to reduce the risk of unauthorized access to critical cloud infrastructure
- Compliance Officers making sure their organization's AWS footprint meets security best practices and standards

## What is the risk?

The main risk here is that a bad actor could be trying to brute force their way into your AWS account by hammering away at the login page hoping to guess a user's password. And you might have no idea it's even happening!

Monitoring failed console login attempts can help provide early indication of an attack in progress. You can use this info to block offending IP addresses, force password resets, notify users of suspicious activity on their accounts, and generally ruin an attacker's day.

## What's the care factor?

On a scale from "meh" to "everybody panic!", monitoring console sign-in failures rates about a 7 or 8. It may not be the absolute highest priority compared to other security tasks, but it's still pretty darn important, especially for large or sensitive AWS environments.

Brute force attacks are a real risk and still a commonly used tactic. Credential compromise is bad news bears. At a minimum, attacks are a nuisance that could lock out legitimate users. Worst case, a successful attack could allow complete takeover of your AWS account. 

## When is it relevant?

This control makes sense for pretty much every AWS account, unless you've completely disabled console access for everyone. The more users and the more sensitive the workloads, the more important it becomes.

About the only situation where you might skip it is for a small, isolated lab or test environment with no real data or resources attackers would care about. But for any production environment, especially in regulated industries, monitoring failed logins is a must-have.

## What are the trade offs?

As with many security practices, logging and monitoring isn't free. It will take some initial engineering effort to set up, and there will be some minor ongoing costs for log storage, processing, and alerting. There's also the risk of alert fatigue if thresholds aren't tuned well.

But in the grand scheme of things, the costs are pretty minimal, especially when you consider the risks of NOT monitoring. And there are ways to be smart about it, like limiting retention time for logs you don't need long-term, and carefully choosing alert thresholds to avoid excessive noise.

## How to make it happen?

Okay, brass tacks time. Here's how you set this up in your AWS environment:

1. Make sure you have CloudTrail enabled to capture management events and send logs to CloudWatch Logs. Create a new trail if needed:

```
aws cloudtrail create-trail --name my-trail --s3-bucket-name my-bucket
```

2. Create a CloudWatch Logs Metric Filter to look for failed console logins and trigger an alarm:

```
aws logs put-metric-filter \
--log-group-name <cloudtrail_log_group_name> \
--filter-name ConsoleSignInFailureMetric \  
--metric-transformations \
metricName=ConsoleSignInFailureCount,metricNamespace=CISBenchmark,metricValue=1 \
--filter-pattern '{ ($.eventName = ConsoleLogin) && ($.errorMessage = "Failed authentication") }'
```

3. Create an SNS topic to notify when the alarm is triggered:

```   
aws sns create-topic --name AccountActivityAlerts
```

4. Subscribe your email to the SNS topic:

```
aws sns subscribe \
--topic-arn arn:aws:sns:us-east-1:123456789012:AccountActivityAlerts \
--protocol email \
--notification-endpoint you@yourdomain.com  
```

5. Finally, create the CloudWatch Alarm:

```
aws cloudwatch put-metric-alarm \
--alarm-name ConsoleAuthenticationFailureAlarm \
--alarm-description "Alarm for AWS console authentication failures" \
--metric-name ConsoleSignInFailureCount \
--namespace CISBenchmark \
--statistic Sum \
--period 300 \
--threshold 3 \
--comparison-operator GreaterThanOrEqualToThreshold \
--evaluation-periods 1 \
--alarm-actions arn:aws:sns:us-east-1:123456789012:AccountActivityAlerts
```

This will trigger the alarm and send an email alert whenever there are 3 or more failed console logins in a 5 minute period. Tune the threshold and time period to your liking.

## What are some gotchas?

A few things to keep in mind when setting this up:

- Make sure you have the necessary IAM permissions to create and configure all the resources involved (CloudTrail, CloudWatch, SNS). The key ones are `cloudtrail:CreateTrail`, `logs:PutMetricFilter`, `cloudwatch:PutMetricAlarm`, and `sns:CreateTopic` 
- If you have an existing CloudTrail trail, make sure it's set to capture Management Events from all regions. Use `aws cloudtrail get-event-selectors` to check.
- Be thoughtful about who has access to view and manage the logs, metrics, and alerts. Restrict to only those who really need it.

## What are the alternatives?

This approach uses all native AWS services, which is great. But maybe you already have a different logging and monitoring stack you prefer, like Splunk, SumoLogic, or ELK. 

No worries! You can still ship your CloudTrail logs over to those systems and build the alerts there instead. Most major SIEM and monitoring tools have plugins or integrations to make ingesting CloudTrail logs pretty easy.

## Explore further

Want to learn more? Check out these resources:

- User Guide for AWS CloudTrail: https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-user-guide.html
- CIS AWS Foundations Benchmark - Monitoring section: https://d1.awsstatic.com/whitepapers/compliance/AWS_CIS_Foundations_Benchmark.pdf   
- CloudWatch Logs Metric Filter docs: https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/MonitoringPolicyExamples.html
- Setting CloudWatch Alarms: https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/AlarmThatSendsEmail.html
- CIS Controls v8 - Control 8: Audit Log Management: https://www.cisecurity.org/controls/v8/

That just about does it! Go forth and monitor those authentication failures. May your logs be ever vigilant and your alerts always helpful. Happy securing!