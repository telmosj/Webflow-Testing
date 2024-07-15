# 4.10 Ensure security group changes are monitored (Manual)

## Summary
It's critical to keep a close eye on any modifications to your AWS security groups. Unexpected changes could accidentally expose your resources and services to the world, introducing serious security risks. By monitoring security group activity in real-time, you can quickly detect and respond to potential threats.

## Where did this come from?
This recommendation comes from the CIS Amazon Web Services Foundations Benchmark v3.0.0 - 01-31-2024. You can download the full benchmark document from the [CIS website](https://downloads.cisecurity.org/#/). The benchmark provides prescriptive guidance for configuring security options across various AWS services. For more details on security groups, check out the [AWS documentation](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html).

## Who should care? 
* Security engineers responsible for protecting AWS environments
* DevOps teams deploying and managing applications on AWS 
* Compliance officers ensuring adherence to security best practices
* IT leaders accountable for the security posture of their organization

## What is the risk?
Uncontrolled changes to security groups could inadvertently allow inbound traffic from malicious sources or outbound connections to command-and-control servers. This opens the door to:
* Data breaches from unauthorized access to sensitive information
* Disruption of services due to resource exhaustion attacks
* Reputation damage and financial losses from compliance violations

While monitoring security groups won't eliminate these risks entirely, it provides critical visibility needed to identify suspicious activity and take prompt action. The sooner you can detect and block an attack, the lower the impact.

## What's the care factor?
On a scale of 1-10, security group monitoring is a solid 8. It's a fundamental security control that every AWS environment should have in place. The risks of not monitoring are simply too high to ignore. However, it's not a complete solution on its own. You still need additional layers of defense like patch management, access control, encryption, etc. But security group monitoring sets a strong foundation to build upon.

## When is it relevant?
Security group monitoring makes sense for any AWS environment, regardless of size or industry. It's especially important for:
* Regulated sectors like finance, healthcare, government 
* Organizations handling sensitive personal data
* High-traffic web applications and APIs
* DevOps teams frequently deploying updates

There may be some exceptions for extremely locked down environments with no external connectivity, or test/dev systems with no production data. But as a general rule, if a resource is important enough to put in a VPC, it's important enough to monitor.

## What are the trade-offs?
Proper security group monitoring does require some effort to set up and maintain:
* Increased data ingestion and storage costs for CloudTrail logs 
* Potential alert fatigue from false positives needing to be tuned
* Ongoing tweaking of alarms as environment evolves over time

However, these costs are minor compared to the risks of a major security incident. You're far better off investing in prevention than cleaning up after an attack.

## How to make it happen?
For real-time security group monitoring, you'll need to set up CloudTrail with CloudWatch Logs integration. Here's how:

1. Turn on CloudTrail in all regions and point its logs to CloudWatch.
2. Create a new CloudWatch Logs Metric Filter to detect security group changes.
    ```
    aws logs put-metric-filter \
      --log-group-name "my-log-group" \  
      --filter-name "SecurityGroupEventCount" \
      --metric-transformations \
        metricName=SecurityGroupEventCount,metricNamespace=CISBenchmark,metricValue=1 \
      --filter-pattern '{ ($.eventName = AuthorizeSecurityGroupIngress) || 
                          ($.eventName = AuthorizeSecurityGroupEgress) ||
                          ($.eventName = RevokeSecurityGroupIngress) || 
                          ($.eventName = RevokeSecurityGroupEgress) || 
                          ($.eventName = CreateSecurityGroup) || 
                          ($.eventName = DeleteSecurityGroup) }'
    ```
3. Create an SNS topic and subscription to receive email/SMS alerts
    ```
    aws sns create-topic --name SecurityGroupChanges
    aws sns subscribe \
      --topic-arn arn:aws:sns:us-west-2:123456789012:SecurityGroupChanges \
      --protocol email \
      --notification-endpoint my-email@example.com
    ```
4. Set up a CloudWatch Alarm to notify on security group events
    ```
    aws cloudwatch put-metric-alarm \
      --alarm-name SecurityGroupChangesAlarm \
      --alarm-description "Alert on security group changes" \
      --metric-name SecurityGroupEventCount \
      --namespace CISBenchmark \
      --statistic Sum \
      --period 300 \
      --threshold 1 \
      --comparison-operator GreaterThanOrEqualToThreshold \
      --evaluation-periods 1 \
      --alarm-actions arn:aws:sns:us-west-2:123456789012:SecurityGroupChanges 
    ```

## What are some gotchas?
A few things to watch out for when implementing security group monitoring:

* Filtering granularity - double check your metric filter regex pattern to avoid blind spots
* Permissions - ensure the IAM role running the CLI commands has the necessary permissions:
    - `logs:PutMetricFilter`
    - `logs:DescribeMetricFilters`  
    - `sns:CreateTopic`
    - `sns:Subscribe`
    - `cloudwatch:PutMetricAlarm`
    - `cloudwatch:DescribeAlarms` 
* Regional vs global resources - security groups are tied to a single region, but CloudTrail is global by default. Disable it in unused regions to limit storage costs.

## What are the alternatives?
AWS Config Rules is another way to detect security group changes. It evaluates recorded configurations against desired settings and sends notifications if they differ. You could write a custom Config Rule in AWS Lambda to check security groups and alert on any modifications. The upside is simpler implementation, but the tradeoff is it's not real-time like CloudTrail.

## Explore further
* Deep dive on AWS Security Groups - [docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html)
* CloudTrail integration with CloudWatch Logs - [docs.aws.amazon.com/awscloudtrail/latest/userguide/send-cloudtrail-events-to-cloudwatch-logs.html](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/send-cloudtrail-events-to-cloudwatch-logs.html)  
* Creating CloudWatch Alarms - [docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/AlarmThatSendsEmail.html](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/AlarmThatSendsEmail.html)
* CIS AWS Foundations Benchmark Recommendations:
    - 4.1 Ensure a log metric filter and alarm exist for unauthorized API calls
    - 4.2 Ensure a log metric filter and alarm exist for Management Console sign-in without MFA
    - 4.4 Ensure a log metric filter and alarm exist for IAM policy changes

This recommendation relates to [CIS Control 3.3](https://www.cisecurity.org/controls/cis-controls-navigator) which advises configuring data access control lists, as well as 8.11 on conducting audit log reviews. It also maps to control 6.2 on activating audit logging and 14.6 on protecting information through access control lists from the CIS Controls v7.