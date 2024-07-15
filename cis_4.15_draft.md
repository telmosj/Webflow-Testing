# 4.15 Ensure AWS Organizations changes are monitored (Manual)

AWS Organizations is a powerful service that lets you centrally manage and govern your AWS environment across multiple accounts. But with great power comes great responsibility. It's critical to keep a close eye on any changes made to your organization to quickly detect and respond to potential security incidents.

In this article, we'll explore why monitoring AWS Organizations is so important, who should care about it, and most importantly - the exact steps you need to take to make it happen using CloudTrail, CloudWatch Logs, and SNS. Let's dive in!

## Where did this come from?

This recommendation comes from the [CIS Amazon Web Services Foundations Benchmark v1.4.0](https://downloads.cisecurity.org/#/). Specifically, section 4.15 which advises monitoring AWS Organizations activity for security best practices.

The CIS AWS Foundations Benchmark provides prescriptive guidance for configuring security options for a subset of Amazon Web Services with an emphasis on foundational, testable, and architecture agnostic settings. [AWS also provides guidance](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_security_incident-response.html) on securing AWS Organizations.

## Who should care? 

This is most relevant for:

- Cloud Security Architects looking to improve the security posture of their AWS environment
- Cloud Operations Engineers responsible for implementing security best practices 
- Governance, Risk, and Compliance (GRC) Managers needing to demonstrate security controls to auditors and meet compliance requirements
- CISOs and Security Leadership accountable for the overall security of the AWS environment

## What is the risk?

AWS Organizations is the central nervous system of your multi-account environment. An attacker with sufficient privileges could use it to:

- Create new accounts and use them for malicious activity
- Modify policies to grant unintended permissions
- Exfiltrate data from attached accounts
- Remove accounts from the organization to evade governance controls
- Delete the entire organization causing a severe business disruption

While hopefully unlikely, the impact of these types of attacks could be catastrophic. Monitoring can help detect malicious activity early to limit the blast radius.

## What's the care factor?

For most organizations, the care factor for this should be high. The security of your entire cloud environment depends on it. 

However, the actual priority depends on your risk tolerance and threat model. For example, if you are an early-stage startup with a simple single-account architecture, it may be lower priority than other security initiatives. But for a large enterprise with a complex multi-account environment, it should be one of the first things you implement.

## When is it relevant?

This recommendation is relevant for almost any non-trivial AWS environment leveraging multiple accounts and AWS Organizations. It's especially critical for organizations with a sizable footprint, sensitive data, or serving highly regulated industries.

However, it may be overkill for very simple environments, POCs, or non-production development. Use your judgment based on your architecture and risk profile.

## What are the trade-offs? 

The main cost is the effort required to configure the monitoring and respond to alerts. You'll need to spend time:

- Setting up CloudTrail with proper logging 
- Configuring CloudWatch Logs metric filters and alarms
- Creating SNS topics and subscriptions for notifications
- Writing play books to triage and investigate alerts
- Training your teams on how to respond 

There is also the cost of storing the logs and potential SNS charges, but this is usually negligible compared to the security benefits.

The alerting thresholds may also need some tuning to avoid alert fatigue from false positives. But you're better off starting broad and then narrowing, rather than potentially missing an incident.

## How to make it happen?

Here are the detailed steps to implement monitoring for AWS Organizations changes:

1. Ensure you have an active multi-region CloudTrail. To check:
   ```
   aws cloudtrail describe-trails
   ```

   Look for a trail with `IsMultiRegionTrail` set to `true`. Note the trail name and `<cloudtrail_log_group_name>` associated with `CloudWatchLogsLogGroupArn`.

2. Ensure the CloudTrail is logging management events:
   ```
   aws cloudtrail get-event-selectors --trail-name <trail name>
   ```

   There should be an event selector with `IncludeManagementEvents` set to `true` and `ReadWriteType` set to `All`.

3. Create a CloudWatch Logs metric filter:
   ```
   aws logs put-metric-filter \
     --log-group-name <cloudtrail_log_group_name> \
     --filter-name OrgsChanges \
     --metric-transformations \
         metricName=OrgsChanges,metricNamespace='CISBenchmark',metricValue=1 \
     --filter-pattern '{ ($.eventSource = organizations.amazonaws.com) && (($.eventName = "AcceptHandshake") || ($.eventName = "AttachPolicy") || ($.eventName = "CreateAccount") || ($.eventName = "CreateOrganizationalUnit") || ($.eventName = "CreatePolicy") || ($.eventName = "DeclineHandshake") || ($.eventName = "DeleteOrganization") || ($.eventName = "DeleteOrganizationalUnit") || ($.eventName = "DeletePolicy") || ($.eventName = "DetachPolicy") || ($.eventName = "DisablePolicyType") || ($.eventName = "EnablePolicyType") || ($.eventName = "InviteAccountToOrganization") || ($.eventName = "LeaveOrganization") || ($.eventName = "MoveAccount") || ($.eventName = "RemoveAccountFromOrganization") || ($.eventName = "UpdatePolicy") || ($.eventName = "UpdateOrganizationalUnit")) }'  
   ```

   This will send a metric value of 1 every time an AWS Organizations API action is logged. You can customize the `metricName` and `metricNamespace` if desired.

4. Create an SNS topic for notifications:
   ```
   aws sns create-topic --name OrgChanges
   ```

   Note the returned `<sns_topic_arn>` value.

5. Subscribe your desired endpoints (email, lambda, etc.) to the SNS topic:
   ```
   aws sns subscribe \
     --topic-arn <sns_topic_arn> \
     --protocol <protocol> \
     --notification-endpoint <endpoint>
   ```

6. Create a CloudWatch alarm:
   ```
   aws cloudwatch put-metric-alarm \
     --alarm-name OrgChanges \
     --metric-name OrgChanges \
     --statistic Sum \
     --period 300 \
     --threshold 1 \
     --comparison-operator GreaterThanOrEqualToThreshold \
     --evaluation-periods 1 \
     --namespace 'CISBenchmark' \
     --alarm-actions <sns_topic_arn>
   ```

   This will trigger the alarm and send an SNS notification whenever the `OrgChanges` metric breaches the threshold of 1 during a 5-minute period.

## What are some gotchas?

Some things to watch out for when implementing this:

- Ensure the IAM role for CloudTrail has sufficient permissions to create and write to the CloudWatch Logs log group. It needs at least the `logs:CreateLogStream` and `logs:PutLogEvents` actions.

- If you have an existing CloudTrail, be careful not to disable it or modify any settings when checking/configuring it for this. You don't want to inadvertently stop logging.

- The SNS topic policy needs to allow CloudWatch to publish to it. When you create the alarm, it should automatically update the policy, but double-check.

- The alarm threshold and period may need tuning depending on your typical activity and risk appetite. You may want to start with a lower threshold and gradually increase it to avoid false positives.

- Make sure you have a process in place to actually monitor and respond to the alerts. An unmonitored inbox won't do you any good.

## What are the alternatives?

Some alternative approaches to monitoring AWS Organizations changes include:

- Sending the CloudTrail logs to a third-party SIEM like Splunk or SumoLogic and configuring alerts there
- Using AWS Config rules to detect and alert on configuration changes
- Using AWS CloudFormation drift detection to identify out-of-band changes 
- Scheduling a script to periodically check your organization's configuration using the `aws organizations describe-*` CLI commands

Each has its pros and cons in terms of capabilities, flexibility, and ease.