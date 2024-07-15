# 4.4 Ensure IAM policy changes are monitored (Manual)

## Summary
It's crucial to keep a close eye on any modifications made to your AWS Identity and Access Management (IAM) policies. By setting up real-time monitoring using AWS CloudTrail and CloudWatch, you can quickly detect and respond to unauthorized changes that could potentially compromise the security of your AWS environment. Trust me, it's worth the effort to set this up!

## Where did this come from?
This recommendation comes straight from the big brains over at the Center for Internet Security (CIS) in their CIS Amazon Web Services Foundations Benchmark v3.0.0 - 01-31-2024 document. You can download the full benchmark from the CIS website at https://downloads.cisecurity.org/#/. The benchmark provides a wealth of guidance on how to securely configure your AWS environment. For more juicy details on monitoring IAM policy changes, check out the AWS documentation at https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudwatch-alarms-for-cloudtrail.html.

## Who should care?
This is a must-read for:
- Cloud Security Architects looking to lock down their AWS environment
- DevOps Engineers responsible for maintaining a secure AWS infrastructure 
- Compliance Officers who need to ensure adherence to security best practices

## What is the risk?
Imagine this nightmare scenario: a rogue administrator or compromised account quietly modifies IAM policies, granting themselves or others excessive permissions. They could then wreak havoc in your environment, exfiltrate sensitive data, or even rack up a massive bill on your dime. Scared yet? By monitoring IAM policy changes, you can quickly identify and mitigate these types of threats before they cause serious damage.

## What's the care factor?  
On a scale from "meh" to "drop everything and fix this now", monitoring IAM policy changes rates a solid "seriously, do this ASAP". While there may be some false positives to tune out, the peace of mind knowing that you'll catch any unauthorized IAM changes is well worth the effort. Don't be the next headline about a preventable data breach!

## When is it relevant?
You should absolutely monitor IAM policy changes if:
- You have a complex AWS environment with many users and roles
- You're subject to compliance requirements like PCI-DSS, HIPAA, or SOC2 
- You value the security of your data and systems (which should be everyone)

There may be some situations where the juice isn't worth the squeeze, like if you have a super simple setup with only a single admin. But in general, this is a best practice that every AWS shop should follow.

## What are the trade offs?
As with any security control, there are some costs to consider:
- You'll need to invest some time upfront to configure the CloudTrail trails, CloudWatch alarms, and SNS notifications 
- Someone will need to monitor the alerts and investigate any suspicious activity
- Depending on your environment, you may get some noise in the form of false positives that need tuning

However, these costs pale in comparison to the potential damage of an IAM policy change going unnoticed. And once you have it set up, it's pretty low effort to maintain.

## How to make it happen?
Alright, let's get down to brass tacks. Here's how to set this up:

1. Make sure you have an active multi-region CloudTrail that captures management events. You can check this with:

```
aws cloudtrail describe-trails
aws cloudtrail get-trail-status --name <trail-name>  
aws cloudtrail get-event-selectors --trail-name <trail-name>
```

2. Create a new CloudWatch Logs metric filter on your CloudTrail log group:

```
aws logs put-metric-filter \
  --log-group-name <cloudtrail-log-group> \
  --filter-name IAMPolicyChanges \
  --metric-transformations metricName=IAMPolicyChanges,metricNamespace=CISBenchmark,metricValue=1 \
  --filter-pattern '{($.eventName=DeleteGroupPolicy)||($.eventName=DeleteRolePolicy)||($.eventName=DeleteUserPolicy)||($.eventName=PutGroupPolicy)||($.eventName=PutRolePolicy)||($.eventName=PutUserPolicy)||($.eventName=CreatePolicy)||($.eventName=DeletePolicy)||($.eventName=CreatePolicyVersion)||($.eventName=DeletePolicyVersion)||($.eventName=AttachRolePolicy)||($.eventName=DetachRolePolicy)||($.eventName=AttachUserPolicy)||($.eventName=DetachUserPolicy)||($.eventName=AttachGroupPolicy)||($.eventName=DetachGroupPolicy)}'
```

This gnarly looking filter pattern will match any API event that modifies IAM policies.

3. Create an SNS topic and subscription to receive the alerts:

```
aws sns create-topic --name CISAlarms
aws sns subscribe --topic-arn <topic-arn> --protocol <email|sms> --notification-endpoint <email|phone-number>  
```

4. Finally, create the CloudWatch alarm:

```
aws cloudwatch put-metric-alarm \
  --alarm-name IAMPolicyChanges \
  --metric-name IAMPolicyChanges \
  --statistic Sum \
  --period 300 \
  --threshold 1 \
  --comparison-operator GreaterThanOrEqualToThreshold \
  --evaluation-periods 1 \
  --namespace CISBenchmark \
  --alarm-actions <sns-topic-arn>
```

This will trigger the alarm and send a notification whenever any IAM policy change events occur within a 5 minute period.

## What are some gotchas?
A few things to watch out for when setting this up:
- Make sure your CloudTrail is configured as a multi-region trail to capture events from all regions
- The CloudWatch alarm requires the `cloudwatch:PutMetricAlarm` permission, and the SNS topic and subscription require `sns:CreateTopic` and `sns:Subscribe` respectively 
- If you're using an existing SNS topic, make sure the necessary IAM permissions are in place for CloudWatch to publish to it

## What are the alternatives?
While CloudTrail and CloudWatch are the standard approach, there are some other options to consider:
- You could stream the CloudTrail events to a 3rd party SIEM like Splunk or SumoLogic to do the alerting 
- Some configuration management tools like AWS Config or HashiCorp Terraform have the ability to detect and alert on changes
- Preventative controls like Service Control Policies (SCPs) can be used to restrict IAM permissions and changes at the AWS Organization level

## Explore further
Here are some additional resources to learn more:
- CIS AWS Foundations Benchmark: https://www.cisecurity.org/benchmark/amazon_web_services/  
- AWS CloudTrail Documentation: https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-user-guide.html
- AWS CloudWatch Documentation: https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/WhatIsCloudWatch.html
- Automating CIS AWS Benchmark checks with AWS Security Hub: https://aws.amazon.com/about-aws/whats-new/2022/11/security-hub-center-internet-securitys-cis-foundations-benchmark-version-1-4-0/

This CIS recommendation aligns with CIS Control 16 "Account Monitoring and Control" which stresses the importance of actively monitoring and controlling user accounts and their activities.