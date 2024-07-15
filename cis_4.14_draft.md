# 4.14 Ensure VPC changes are monitored (Manual)

Creating and modifying Virtual Private Clouds (VPCs) in AWS is a breeze, but are you keeping a close eye on those changes? You should be! Unauthorized VPC modifications can seriously mess with your network accessibility and traffic flow. But fear not, we've got your back with this handy guide on monitoring VPC changes like a pro.

## Where did this come from?

This sage advice comes straight from the "CIS Amazon Web Services Foundations Benchmark v3.0.0 - 01-31-2024". You can download the full benchmark from the [CIS website](https://downloads.cisecurity.org/#/) for more juicy security recommendations. For more nitty-gritty details on VPCs, check out the [AWS VPC documentation](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html).

## Who should care? 

This matters for:
- Cloud engineers with a need to keep the network secure and humming 
- Security analysts with a desire to detect suspicious VPC changes
- Compliance officers with a duty to enforce strict change management 
- IT managers with a responsibility to control cloud costs and usage

## What is the risk?

Unauthorized VPC changes can lead to a world of pain:
- Accidentally exposing private resources to the public internet 
- Breaking network connectivity for critical applications
- Enabling data exfiltration via VPC peering 
- Incurring surprise costs from unintended VPC usage

This CIS recommendation helps you prevent and quickly detect VPC misconfigurations that open security holes. Timely alerts allow you to investigate and revert unwanted changes before bad hats exploit them.

## What's the care factor?

On a scale from "meh" to "code red", improper VPC changes rate at least an orange alert. While not necessarily an emergency, unapproved VPC modifications are a slippery slope to security incidents and compliance problems. Monitoring changes shows due diligence and allows you to keep tabs on "shadow IT" activity.

High-stakes applications and sensitive data justify an even higher alerting priority for VPC events. So crank it up to red if you're hosting anything mission-critical or regulated in your VPCs.

## When is it relevant?

Monitoring VPC changes makes sense whenever you:
- Use a VPC (obviously)
- Want to enforce change management processes
- Need to maintain strict network isolation  
- Must comply with regulatory standards
- Are concerned about VPC sprawl and "shadow IT"

On the flip side, tiny sandbox accounts or test environments may not warrant the same level of scrutiny. Use your judgment based on risk.

## What are the trade-offs?

On the plus side, VPC monitoring is easy to set up with native AWS tools like CloudTrail and CloudWatch. Just toggle on some settings and you're good to go. The only real "costs" are:

- A few minutes of initial configuration time
- The effort to triage and investigate alerts
- Potentially bugging developers to submit change requests
- Infinitesimally higher AWS logging charges

This is a small price to pay for the warm fuzzy feeling of knowing your VPCs aren't running wild!

## How to make it happen?

Time to walk the walk. Here's how to set up VPC monitoring, step by step:

1. Turn on CloudTrail in all regions. Be sure to enable "Management Events" so it logs VPC control-plane operations.

2. Create a CloudWatch Logs metric filter to catch VPC change events:

   Filter pattern:
   ```
   { ($.eventName = CreateVpc) || ($.eventName = DeleteVpc) || ($.eventName = ModifyVpcAttribute) || ($.eventName = AcceptVpcPeeringConnection) || ($.eventName = CreateVpcPeeringConnection) || ($.eventName = DeleteVpcPeeringConnection) || ($.eventName = RejectVpcPeeringConnection) || ($.eventName = AttachClassicLinkVpc) || ($.eventName = DetachClassicLinkVpc) || ($.eventName = DisableVpcClassicLink) || ($.eventName = EnableVpcClassicLink) }
   ```
   
   Metric name: `VpcEventCount`
   Metric namespace: `CISBenchmark`
   Metric value: `1`

3. Set up a CloudWatch alarm on the `VpcEventCount` metric:

   Alarm name: `VpcChangesDetected` 
   Threshold: `>= 1` (trigger on any event)
   Period: 5 minutes
   Statistic: Sum

4. Have the alarm send notifications to an SNS topic.

   Topic name: `VpcChangeAlerts`

5. Subscribe your team's email addresses to the SNS topic.

That's it! Now you'll get an email alert whenever someone creates, modifies, or deletes a VPC.

## What are some gotchas?

To set up VPC monitoring, you'll need permissions to:
- Enable CloudTrail logging (`cloudtrail:UpdateTrail`, `cloudtrail:PutEventSelectors`)
- Create CloudWatch Logs metric filters (`logs:PutMetricFilter`) 
- Set CloudWatch alarms (`cloudwatch:PutMetricAlarm`)
- Manage SNS topics and subscriptions (`sns:CreateTopic`, `sns:Subscribe`)

Your IAM policies must allow these specific actions. Check the [CloudTrail](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/security_iam_id-based-policy-examples.html), [CloudWatch Logs](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/iam-identity-based-access-control-cwl.html), and [SNS](https://docs.aws.amazon.com/sns/latest/dg/UsingIAMwithSNS.html) docs for details.

Also, keep in mind that this setup only tracks changes to VPC *resources*. To monitor actual network traffic flowing through the VPC, you'll need additional tools like VPC Flow Logs.

## What are the alternatives?

Instead of relying on manual email alerts, you could send CloudTrail events to a third-party SIEM platform for fancier automated response options. Vendors like Splunk, SumoLogic, and AlertLogic have tight integrations with AWS.

For a hipster option, pipe the CloudTrail logs to an ELK stack and craft your own custom dashboards and alerts. Open-source intrusion detection systems like Wazuh can augment basic VPC monitoring with active defense.

## Explore further

Hungry for more? Dig deeper with:
- [VPC Security](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Security.html) - The official AWS docs on locking down your VPCs
- [CIS Benchmark](https://downloads.cisecurity.org/#/) - Get the full 400+ page guide

This recommendation aligns with:
- CIS Control 8.11 - Conduct Audit Log Reviews
- CIS Control 5.5 - Implement Automated Configuration Monitoring Systems

Stay vigilant and keep those VPCs in check! Happy monitoring!