# 4.12 Ensure changes to network gateways are monitored

## Summary
It's important to keep a close eye on your network gateways in AWS. These are the entry and exit points for traffic flowing in and out of your VPCs. Any unexpected changes could open up security holes or disrupt connectivity. Luckily, AWS provides tools to monitor gateway configuration changes in real-time.

## Where did this come from?
This recommendation comes from the CIS Amazon Web Services Foundations Benchmark v3.0.0 published on 01-31-2024. You can download the full benchmark from the [CIS website](https://downloads.cisecurity.org/#/). The document provides prescriptive guidance for configuring AWS account and services to align with security best practices. Check out the [AWS CloudTrail documentation](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-user-guide.html) for more details on monitoring API activity.

## Who should care? 
This matters most to:
- Cloud Security Engineers responsible for securing AWS environments
- DevOps Engineers managing VPC networks and connectivity 
- Cloud Architects designing network topologies
- Compliance Officers ensuring adherence to security standards

## What is the risk?
Undetected changes to network gateways could lead to:
- Exposure of private resources to the internet
- Creation of unintended network paths that bypass security controls
- Outages due to misconfigured or deleted gateways
- Compliance violations 

While monitoring won't prevent all misconfigurations, it provides a detective control to catch issues quickly. The sooner you identify a problem, the faster you can remediate it to limit the blast radius.

## What's the care factor?
On a scale of 1-10, gateway monitoring is a solid 7. It's not quite as critical as making sure your root account is secured, but it's still pretty darn important. VPC traffic traverses these gateways, so you need to know if anything fishy is going on. A single misconfigured gateway could jeopardize your whole environment.

## When is it relevant?
You should enable gateway monitoring in virtually all scenarios when using AWS VPCs, which is pretty much always. The only potential exception would be a standalone VPC with no external connectivity requirements. But even then, it's still a good safety net.

## What are the trade offs?
The main cost is the effort required to configure the CloudWatch Logs metric filters, alarms, and SNS notifications. It's not a huge lift, but it does take some clicking around in the AWS Console or commandeering the AWS CLI. You also have to keep an eye on the alerts and tune them over time to avoid alert fatigue.

Sending the CloudTrail logs to CloudWatch will incur some minor costs for log ingestion and storage. But it's money well spent for the security visibility. Just be sure to configure lifecycle policies to archive/expire old logs.

## How to make it happen?
We're going assume you already have CloudTrail enabled to capture management events in all regions. If not, start there first. Then follow these steps:

1. Open the CloudWatch Console and navigate to Logs > Log groups.
2. Click Actions and select "Create Metric Filter"
3. Under "Filter Pattern", enter: 
  ```
  { ($.eventName = CreateCustomerGateway) || ($.eventName = DeleteCustomerGateway) || ($.eventName = AttachInternetGateway) || ($.eventName = CreateInternetGateway) || ($.eventName = DeleteInternetGateway) || ($.eventName = DetachInternetGateway) }
  ```
4. Click "Assign Metric" and enter a name like `GatewayChanges`. Set the value to 1.
5. Click "Create Filter". You should see the new filter listed.
6. Now head over to the CloudWatch Alarms section and click "Create Alarm".  
7. Select the `GatewayChanges` metric and set the alarm threshold to >=1 for 1 period of 5 minutes.
8. Under "Actions", create a new SNS topic for the notifications, or choose an existing one.
9. Name the alarm something like `NetworkGatewayChanges` and click "Create Alarm".

You're all set! You'll now get an email/SMS anytime someone makes a change to a gateway. Be sure to test it to make sure it's working.

## What are some gotchas?
- Make sure the IAM role/user creating the alarm has the `cloudwatch:PutMetricAlarm` and `cloudtrail:DescribeTrails` permissions. Easy to overlook!
- Double-check that your management events are getting captured in CloudTrail. The metric filter won't match if the API activity isn't in the logs. 
- There's a small chance of false positives if you've excluded the gateway events from CloudTrail with an event selector. 
- Avoid using periods in your metric name. CloudWatch will reject it.

## What are the alternatives?
You could ship the CloudTrail logs to a 3rd party SIEM (e.g. Splunk) and configure alerting there instead. Some may prefer this route if they are already aggregating logs in an external system. AWS Config is another option that can track gateway config changes, but it lacks real-time alerting.

## Explore further
- Read the [AWS docs on creating CloudWatch alarms](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/AlarmThatSendsEmail.html)
- Check out [this blog post](https://aws.amazon.com/blogs/security/how-to-receive-alerts-when-your-iam-configuration-changes/) for a deep dive on the CloudTrail + CloudWatch Logs integration 
- Consider enabling these other related CIS benchmark recommendations:
    - 3.7 - Ensure VPC flow logging is enabled in all VPCs
    - 5.4 - Ensure the default security group of every VPC restricts all traffic

By following this guide, you'll have taken an important step towards improving the security posture of your AWS environment. Remember, network security is a continuous journey. Stay vigilant my friend! And maybe consider adding "wrote an epic article on AWS network gateway monitoring" to your resume. I won't tell anyone you had a little help from yours truly. 😉