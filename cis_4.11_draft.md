# 4.11 Ensure Network Access Control Lists (NACL) changes are monitored (Manual)

## Summary

Network Access Control Lists (NACLs) are an important security control that act as virtual firewalls for controlling traffic in and out of subnets within an AWS VPC. Changes to NACLs should be closely monitored to ensure they remain properly configured to only allow expected traffic. Fortunately, AWS provides easy ways to monitor NACL changes in real-time.

## Where did this come from?

This recommendation comes from the CIS Amazon Web Services Foundations Benchmark v3.0.0 released on 01-31-2024. The specific recommendation is 4.11. The full CIS AWS Foundations Benchmark can be downloaded from https://downloads.cisecurity.org/#/. 

For more details on AWS NACLs, see:
- [Network ACLs - AWS Documentation](https://docs.aws.amazon.com/managedservices/latest/userguide/restrict-nacl.html)
- [Monitoring NACLs - AWS Documentation](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-network-acls.html)

## Who should care?

- Cloud Security Engineers responsible for securing AWS environments
- DevOps teams deploying services into VPCs who need to manage network access
- Compliance officers ensuring proper security monitoring is in place
- IT Auditors validating security controls are operating effectively

## What is the risk?  

Unintended changes to NACLs can expose AWS resources and services by allowing undesired inbound or outbound traffic. This could enable:

- Unauthorized access to resources from external networks
- Data exfiltration from compromised resources 
- Lateral movement between internal subnets that should be isolated

While NACLs add defense-in-depth, improper rules could allow complete compromise of resources in a VPC. Monitoring changes is critical to detect misconfigurations quickly.

## What's the care factor?

For most organizations, monitoring NACLs should be a high priority, especially for VPCs containing sensitive workloads. The security risks of misconfigured NACLs are severe. 

Fortunately, enabling monitoring is straightforward and low effort, so it's an easy win. There is really no excuse for not having this control in place.

## When is it relevant?

Monitoring NACL changes is relevant for:

- All custom VPCs containing private resources
- Any VPC with subnets that should have restricted access

It's less relevant for:

- Simple VPCs using only the default NACL 
- Public subnets where wide open inbound access is expected

## What are the trade-offs?

Implementing this control is low cost. It requires:

- Enabling a multi-region CloudTrail 
- Configuring a CloudWatch Logs metric filter and alarm
- Someone to respond to alerts

The main ongoing cost is dealing with alerts, which should be minimal since NACL changes are usually infrequent. 

There are no significant performance, functionality, or usability trade-offs. Monitoring is transparent to end users.

## How to make it happen?

The high-level steps are:

1. Enable a multi-region CloudTrail trail 
2. Create a CloudWatch Logs group and configure CloudTrail to send logs to it
3. Create a CloudWatch Logs metric filter to detect NACL events
4. Create a CloudWatch alarm to alert on the metric filter
5. Create an SNS topic to receive alarm notifications
6. Subscribe your team to the SNS topic

Here are the detailed commands to implement it using the AWS CLI:

1. Create the CloudTrail (replace `TRAIL_NAME` with an appropriate name)

```bash
aws cloudtrail create-trail --name TRAIL_NAME --is-multi-region-trail --is-organization-trail
```

2. Create the CloudWatch Logs group (replace `LOG_GROUP_NAME`)

```
aws logs create-log-group --log-group-name LOG_GROUP_NAME
```

3. Create the CloudWatch Logs metric filter (replace `METRIC_NAMESPACE` and `METRIC_NAME`)

```
aws logs put-metric-filter \
  --log-group-name LOG_GROUP_NAME \
  --filter-name NACL_CHANGES \
  --metric-transformations \
      metricName=METRIC_NAME,metricNamespace=METRIC_NAMESPACE,metricValue=1 \
  --filter-pattern \
    '{ ($.eventName = CreateNetworkAcl) || 
       ($.eventName = CreateNetworkAclEntry) ||
       ($.eventName = DeleteNetworkAcl) || 
       ($.eventName = DeleteNetworkAclEntry) ||
       ($.eventName = ReplaceNetworkAclEntry) || 
       ($.eventName = ReplaceNetworkAclAssociation) }'
```

4. Create the CloudWatch alarm (replace `ALARM_NAME`, `METRIC_NAME`, and `METRIC_NAMESPACE`)

```
aws cloudwatch put-metric-alarm \
  --alarm-name ALARM_NAME \
  --alarm-description "Alert on NACL changes" \
  --metric-name METRIC_NAME \
  --namespace METRIC_NAMESPACE \
  --statistic Sum \
  --period 300 \
  --threshold 1 \
  --comparison-operator GreaterThanOrEqualToThreshold \
  --evaluation-periods 1 \
  --unit Count \
  --actions-enabled 
```

5. Create the SNS topic for notifications (replace `TOPIC_NAME`) 

```
aws sns create-topic --name TOPIC_NAME
```

6. Subscribe to the SNS topic (replace `TOPIC_ARN` and `EMAIL_ADDRESS`)

```
aws sns subscribe \
  --topic-arn TOPIC_ARN \
  --protocol email \
  --notification-endpoint EMAIL_ADDRESS
```

## What are some gotchas?

The main gotcha is ensuring you have the necessary IAM permissions to set this all up, including:

- `cloudtrail:CreateTrail`
- `cloudtrail:PutEventSelectors`
- `cloudtrail:StartLogging`
- `logs:CreateLogGroup` 
- `logs:PutMetricFilter`
- `cloudwatch:PutMetricAlarm`

You'll also need to ensure the IAM role for CloudTrail has permission to write to CloudWatch Logs.

Additionally, keep in mind:

- The metric filter will trigger on ANY NACL changes across the entire AWS account
- The alarm doesn't indicate which VPC or NACL was changed, just that some NACL was modified

These may cause false positives that you'll need to dig into. Consider implementing multiple metric filters and alarms for different VPCs if you want more granular alerts.

## What are the alternatives?

Some alternative approaches to monitoring NACL changes include:

- Using the AWS Config service to detect and alert on NACL configuration changes
- Implementing change management database (CMDB) processes around NACL updates
- Periodic manual reviews of NACL configurations
- Implementing infrastructure-as-code and monitoring for divergence from checked-in NACL configurations

However, the CloudTrail + CloudWatch Logs approach is generally the easiest and most real-time method.

## Explore Further

- CIS AWS Foundations Benchmark v3.0.0, Recommendation 4.10 - Ensure security group changes are monitored 
- [AWS CloudTrail Event Reference - CreateNetworkAcl](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-event-reference-management-events.html#createnetworkacl)
- [AWS CloudTrail Event Reference - CreateNetworkAclEntry](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-event-reference-management-events.html#createnetworkaclentry)
- [AWS CloudTrail API Reference - create-trail](https://docs.aws.amazon.com/cli/latest/reference/cloudtrail/create-trail.html)
- [AWS CloudWatch Logs API Reference - put-metric-filter](https://docs.aws.amazon.com/cli/latest/reference/logs/put-metric-filter.html)