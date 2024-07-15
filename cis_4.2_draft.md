# 4.2 Ensure management console sign-in without MFA is monitored (Manual)

## Summary
It's important to keep a watchful eye on your AWS Management Console logins, especially those that don't use multi-factor authentication (MFA). By setting up some nifty monitoring using CloudTrail and CloudWatch, you can get alerted whenever someone manages to sign in without that extra layer of security. Trust me, it's worth the effort to make sure no unauthorized folks are poking around in your AWS environment!

## Where did this come from?
This recommendation comes straight from the CIS Amazon Web Services Foundations Benchmark v3.0.0 - 01-31-2024. You can download the full benchmark document from the [CIS website](https://downloads.cisecurity.org/#/). It's chock-full of other great security tips for locking down your AWS setup. For more background, check out the [AWS CloudTrail documentation](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-user-guide.html) and [CloudWatch Logs documentation](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/WhatIsCloudWatchLogs.html).

## Who should care?
This one is key for:
- AWS administrators who are responsible for securing the AWS environment 
- Security analysts who need to monitor for potential unauthorized access
- Compliance officers who must ensure adherence to security best practices

## What is the risk?
The main risk here is unauthorized access to your AWS environment via the management console. If someone manages to get their hands on valid IAM user credentials, they could sign into the console without MFA and start causing all sorts of mischief. They might view sensitive data, change configurations, or even spin up expensive resources on your dime. Monitoring for non-MFA logins helps you quickly detect and respond to potential compromises.

## What's the care factor?
For most organizations, monitoring console logins without MFA should be a high priority. The AWS Management Console is the central nervous system of your cloud environment. Unauthorized access there is bad news bears. While the likelihood of a breach may be low if you follow other security best practices, the consequences could be severe in terms of data loss, service disruption, and financial cost. Better to set up the monitoring and sleep soundly at night.

## When is it relevant?
This recommendation is relevant for pretty much any AWS environment where there are IAM users signing into the AWS Management Console. The only exception might be if you have a very small setup with just one or two trusted admins and no sensitive data. But even then, it's quick and easy to set up, so might as well do it for the peace of mind.

On the flip side, if you have a complex environment with many accounts and dozens or hundreds of IAM users, monitoring non-MFA console logins is an absolute must. The more users you have, the higher the chances of credentials getting compromised.

## What are the trade offs?
The main "cost" of this control is the time and effort required to set it up initially. But we're talking maybe an hour tops. Once it's in place, it basically runs itself - you just need to make sure someone is paying attention to the alerts. 

There may also be some minor cost in terms of log storage in CloudWatch Logs, but it's usually negligible unless you have an absolutely massive amount of activity. And you can always adjust the retention period to keep costs down.

The benefits of having this monitoring in place far outweigh the minimal costs in most cases. It's one of those low effort, high impact security measures.

## How to make it happen?
Alrighty, let's get down to brass tacks. Here's how you set this up:

1. Make sure you have CloudTrail set up to log to CloudWatch Logs. Specifically, you need a trail that captures management events and has CloudWatch Logs integration enabled. Check the AWS docs for detailed instructions.

2. In the CloudWatch Logs console, create a new metric filter on your CloudTrail log group. Use the following filter pattern:
   ```
   { ($.eventName = "ConsoleLogin") && ($.additionalEventData.MFAUsed != "Yes") && ($.userIdentity.type = "IAMUser") && ($.responseElements.ConsoleLogin = "Success") }
   ```
   This looks for successful console logins where MFA was not used. Name the filter something like `NoMFAConsoleLogins`.

3. On the next screen, select "Create Alarm". Give it a name like "No MFA Console Logins" and a description. Set the threshold to trigger if there is >= 1 login without MFA in a 5 minute period. 

4. Under Actions, select an SNS topic to notify when the alarm triggers. If you don't have one set up already, you can create a new one.

5. Review and create the alarm. That's it! You'll now get an alert whenever someone logs into the console without MFA.

For the CLI fans out there, you can also set this up using the AWS CLI. Just run these commands, filling in the placeholders:

```
aws logs put-metric-filter \
  --log-group-name <cloudtrail_log_group_name> \
  --filter-name NoMFAConsoleSignins \  
  --metric-transformations metricName=NoMFAConsoleSignins,metricNamespace='CISBenchmark',metricValue=1 \
  --filter-pattern '{ ($.eventName = "ConsoleLogin") && ($.additionalEventData.MFAUsed != "Yes") && ($.userIdentity.type = "IAMUser") && ($.responseElements.ConsoleLogin = "Success") }'

aws cloudwatch put-metric-alarm \
  --alarm-name NoMFAConsoleSignins \
  --metric-name NoMFAConsoleSignins \
  --statistic Sum \
  --period 300 \
  --threshold 1 \
  --comparison-operator GreaterThanOrEqualToThreshold \
  --evaluation-periods 1 \ 
  --namespace 'CISBenchmark' \
  --alarm-actions <sns_topic_arn>
```

## What are some gotchas?
There are a few things to keep in mind when implementing this control:

- Make sure you have the right permissions to create the metric filter and alarm. You'll need `logs:PutMetricFilter`, `logs:DescribeMetricFilters`, and `cloudwatch:PutMetricAlarm` permissions. 

- If you use AWS SSO for console access, you may get some false positives since those logins won't have the user identity type of "IAMUser". You can adjust the filter pattern to account for this if needed.

- This control only monitors console logins initiated by an IAM user. It won't catch logins using root credentials, IAM roles, or temporary security tokens. But you can setup other alarms for those cases.

- Keep in mind that this is a detective control, not a preventive one. It won't actually stop someone from logging in without MFA, it will just alert you after the fact. Preventing non-MFA logins requires a different control.

## What are the alternatives?
Another way to monitor console logins is to use the AWS CloudTrail Event History feature in the AWS Management Console. This lets you view and filter events, including console sign-in events. However, it doesn't provide the same real-time alerting as the CloudWatch Logs method.

You could also send your CloudTrail logs to a 3rd party SIEM like Splunk or ELK. Then you can set up alerts in your SIEM system using a similar filter/query for non-MFA logins. This can work well if you're already using a SIEM to centralize your logging and monitoring.

## Explore further
Want to learn more? Check out these resources:

- AWS Documentation: [Creating CloudWatch Alarms for CloudTrail Events](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudwatch-alarms-for-cloudtrail.html) 
- AWS Documentation: [Monitoring Console Sign-in Events](https://docs.aws.amazon.com/IAM/latest/UserGuide/cloudtrail-integration.html#console-sign-in-events)
- Related CIS Benchmarks:
  - 1.2 - Ensure multi-factor authentication (MFA) is enabled for all IAM users that have a console password
  - 3.3 - Ensure a log metric filter and alarm exist for usage of "root" account

Hope this helps you keep a tight leash on those console logins! Remember, MFA is bae. Stay safe out there, friends.