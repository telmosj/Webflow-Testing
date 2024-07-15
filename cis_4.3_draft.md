# 4.3 Ensure usage of 'root' account is monitored (Manual)

## Summary
It's critical to keep a close eye on the root account in your AWS environment. The root account has full privileges and if compromised, an attacker could wreak havoc. By monitoring root account activity, you can quickly detect and respond to any suspicious or unauthorized usage.

## Where did this come from?
This recommendation comes from the CIS Amazon Web Services Foundations Benchmark v3.0.0 released on 01-31-2024. You can download a copy of the full benchmark from the [CIS website](https://downloads.cisecurity.org/#/). The benchmark provides prescriptive guidance for configuring AWS accounts with security best practices. For more background, check out [this overview](https://docs.aws.amazon.com/securityhub/latest/userguide/securityhub-cis-controls.html) from AWS.

## Who should care? 
This is relevant for:
- Cloud Security Engineers responsible for securing AWS environments
- Cloud Operations teams that manage AWS accounts
- Compliance officers that need to demonstrate security monitoring capabilities
- CISOs and security leadership that oversee the cloud security program

## What is the risk?
The root account in AWS has unrestricted access to all resources in the account. If the root credentials are compromised, an attacker could:
- Create, modify or delete any AWS resources 
- Exfiltrate sensitive data stored in AWS
- Rack up a huge bill for unauthorized services
- Cover their tracks by disabling logging 

While the root account should be used sparingly, it's important to have visibility into when and how it is used for security and compliance. This recommendation provides a detective control to identify root activity.

## What's the care factor?
Safeguarding root account usage should be a top priority for any organization using AWS. The impact of a compromised root account is severe. Even if root is not used day-to-day, continuous monitoring is a must-have.

The care factor for this recommendation is very high. It should be implemented in all AWS accounts with alerting and response workflows to handle any exceptions.

## When is it relevant?
This recommendation is relevant for all AWS accounts, all the time. Monitoring root account activity is a foundational practice and should be in place from day one.

The only scenario where it may not apply is for temporary/ephemeral accounts used for testing or demonstrations. But any account handling production systems or real data should absolutely implement root monitoring.

## What are the trade-offs?
Fortunately, there are no major downsides to monitoring the root account. It does not impact performance or the end user experience. 

The only costs are:
1) Minimal spend (pennies per GB) for CloudTrail logging, CloudWatch Logs, and SNS notifications 
2) One-time setup effort to configure the plumbing
3) Occasional time spent triaging and responding to alerts

But these costs are modest compared to the risk reduction benefits. The ROI on root account monitoring is very favorable.

## How to make it happen?
We'll leverage native AWS services to implement root account monitoring. Here's how to set it up:

1. Make sure you have a [multi-region CloudTrail](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/receive-cloudtrail-log-files-from-multiple-regions.html) enabled to capture all management events. Validate it is active and logging:

   ```
   aws cloudtrail describe-trails
   aws cloudtrail get-trail-status --name <trail-name>
   aws cloudtrail get-event-selectors --trail-name <trail-name>
   ```

   Note the CloudWatch log group name that CloudTrail is logging to.

2. [Create a CloudWatch Metric Filter](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/MonitoringLogData.html) to look for root activity:

   ```   
   aws logs put-metric-filter \
     --log-group-name <cloudtrail-log-group> \
     --filter-name RootAccountUsage \  
     --metric-transformations \
         metricName=RootAccountUsage,metricNamespace=CISBenchmark,metricValue=1 \
     --filter-pattern '{ $.userIdentity.type = "Root" && $.userIdentity.invokedBy NOT EXISTS && $.eventType != "AwsServiceEvent" }'
   ```

3. Create an SNS topic that the alarm will notify:

   ```
   aws sns create-topic --name CIS-Alarms-Topic
   ```
   
   Note the topic ARN.

4. Subscribe your email to receive the alerts:

   ```
   aws sns subscribe \
    --topic-arn <CIS-Alarms-Topic-ARN> \
    --protocol email \
    --notification-endpoint <your@email.com>
   ```

5. Create the CloudWatch Alarm: 

   ```
   aws cloudwatch put-metric-alarm \
     --alarm-name RootAccountUsageAlarm \
     --metric-name RootAccountUsage \
     --statistic Sum \
     --period 300 \
     --threshold 1 \
     --comparison-operator GreaterThanOrEqualToThreshold \
     --evaluation-periods 1 \
     --namespace 'CISBenchmark' \
     --alarm-actions <CIS-Alarms-Topic-ARN>
   ```

That's it! You should now receive email alerts whenever the root account is used. Be sure to test it out.

## What are some gotchas?
A few things to keep in mind:

- These steps require permissions to CloudTrail, CloudWatch Logs, SNS. The IAM user/role needs `cloudtrail:*, logs:*, sns:*`

- If you have multiple AWS accounts, you'll need to repeat these steps in each one. Consider using [AWS Organizations](https://aws.amazon.com/organizations/) to help manage multi-account setup.

- You may want to customize the email subject and message body for the SNS subscription to suit your playbook. See the [SNS docs](https://docs.aws.amazon.com/sns/latest/dg/welcome.html).

- In addition to email, you can notify other systems like Slack, PagerDuty, or a SIEM via SNS. Expand your integrations over time.

## What are the alternatives?
The approach outlined here using CloudTrail, CloudWatch and SNS is the standard recommendation from both AWS and CIS.

Some 3rd party CASB (Cloud Access Security Broker) tools like Netskope or Cloudlock may be able to provide AWS root monitoring as well. But for most organizations, the native services are a great fit to get started.

## Explore Further
Want to learn more? Check out these resources:

- [AWS Root User Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html#lock-away-credentials)
- [CIS AWS Foundations Benchmark](https://www.cisecurity.org/benchmark/amazon_web_services/)
- [AWS Security Primer](https://aws.amazon.com/security/introduction-to-aws-cloud-security/)
- [This AWS Security Blog Post](https://aws.amazon.com/blogs/security/how-to-receive-notifications-when-your-aws-accounts-root-access-keys-are-used/) covers a similar setup

Also consider implementing these related recommendations:

- CIS 1.11 - Avoid the use of the "root" account
- CIS 1.12 - Ensure multi-factor authentication (MFA) is enabled for all IAM users that have a console password
- CIS 4.1 - Ensure a log metric filter and alarm exist for unauthorized API calls 
- CIS 4.2 - Ensure a log metric filter and alarm exist for Management Console sign-in without MFA

Hopefully this provides a solid overview of why and how to monitor root account activity in AWS.