# 3.7 Ensure VPC flow logging is enabled in all VPCs (Automated)

## Summary
Hey folks, it's time we had a chat about a super important security feature in AWS - VPC Flow Logs. Think of these logs as the CCTV cameras of your VPC, capturing all the juicy details of the network traffic flowing in and out. Enabling VPC Flow Logs is a must-do if you want to keep an eye out for any fishy activity and stay on top of your security game. Let's dive in and see what the fuss is all about!

## Where did this come from?
This recommendation comes straight from the big shots at the Center for Internet Security (CIS). It's part of their Amazon Web Services Foundations Benchmark v3.0.0, released on 01-31-2024. You can grab a copy of the full benchmark [here](https://downloads.cisecurity.org/#/) and geek out on all the other cool security stuff in there.

Now, if you want to learn more about VPC Flow Logs and how to set them up, AWS has got you covered with their [official documentation](https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs.html). It's chock-full of useful info and step-by-step guides.

## Who should care? 
- **Security analysts** who need visibility into network traffic for detecting anomalies and investigating incidents
- **Network administrators** responsible for monitoring and troubleshooting VPC network issues
- **Compliance officers** ensuring the organization meets regulatory requirements like PCI-DSS, HIPAA, etc.
- **DevOps engineers** setting up and maintaining secure VPC environments

## What is the risk?
Without VPC Flow Logs, you're basically flying blind when it comes to network activity in your VPC. Here are a few scary things that could happen:

1. **Data exfiltration**: An attacker could be siphoning off sensitive data right under your nose, and you wouldn't even know it. VPC Flow Logs can help you spot unusual traffic patterns and investigate potential data leaks.

2. **Malicious activity**: Cybercriminals love to lurk in the shadows of your network, launching attacks and causing mayhem. With VPC Flow Logs, you can detect and respond to malicious traffic before it wreaks havoc.

3. **Compliance violations**: Many regulations require you to monitor and log network activity. Without VPC Flow Logs, you might face hefty fines and damage to your reputation if you fail an audit.

While VPC Flow Logs alone can't prevent these risks, they provide invaluable insights that can help you detect, investigate, and respond to security incidents more effectively.

## What's the care factor?
On a scale of "meh" to "oh my god, we're doomed!", VPC Flow Logs are definitely leaning towards the latter. If you're responsible for the security or compliance of your AWS environment, you should care A LOT about this recommendation.

Why? Because network visibility is the foundation of a strong security posture. You can't protect what you can't see. By enabling VPC Flow Logs, you're giving yourself a fighting chance against the bad guys and demonstrating due diligence to auditors.

Sure, setting up and maintaining VPC Flow Logs takes some effort, but it's a small price to pay for the peace of mind and the ability to detect and respond to threats quickly.

## When is it relevant?
VPC Flow Logs are relevant in almost every AWS environment, but here are some specific situations where they're especially important:

- **High-risk workloads**: If you're running applications that handle sensitive data (e.g., financial, healthcare, personally identifiable information), VPC Flow Logs are a must-have for monitoring access and detecting anomalies.

- **Compliance-regulated industries**: If you're subject to regulations like PCI-DSS, HIPAA, SOC2, etc., VPC Flow Logs can help you meet the logging and monitoring requirements.

- **Multi-tenant environments**: If you're hosting multiple customers or business units in the same VPC, VPC Flow Logs can help you ensure proper network isolation and investigate cross-tenant incidents.

On the flip side, there might be some situations where VPC Flow Logs are less critical, such as:

- **Low-risk, isolated workloads**: If you have a simple, low-risk application running in its own VPC with no external connectivity, the value of VPC Flow Logs might be limited (but still recommended as a best practice).

- **Performance-sensitive workloads**: If you have extremely high-volume, latency-sensitive traffic (e.g., real-time streaming, HPC), the overhead of capturing and storing VPC Flow Logs might impact performance. In such cases, you might consider alternative monitoring techniques or sampling strategies.

## What are the trade-offs?
While enabling VPC Flow Logs is generally a good idea, there are some costs and considerations to keep in mind:

1. **Storage costs**: VPC Flow Logs can generate a ton of data, especially in high-traffic environments. You'll need to pay for the storage of these logs in CloudWatch Logs or S3, which can add up over time. Be sure to set appropriate retention policies and monitor your storage usage closely.

2. **Processing costs**: If you want to analyze your VPC Flow Logs using services like Amazon Athena or ElasticSearch, you'll incur additional processing costs based on the amount of data scanned. 

3. **Performance impact**: Capturing and storing VPC Flow Logs does introduce some overhead, which could potentially impact the performance of your network-intensive workloads. However, in most cases, the impact is negligible and outweighed by the security benefits.

4. **Noise and false positives**: VPC Flow Logs can be noisy, especially in large environments with a lot of traffic. You'll need to tune your analysis and alerting to avoid getting overwhelmed with false positives, which can take some trial and error.

Despite these trade-offs, VPC Flow Logs are still a valuable tool for security monitoring and investigation. The key is to find the right balance between the level of visibility you need and the costs and complexity you can manage.

## How to make it happen?

Alright, let's roll up our sleeves and get VPC Flow Logs set up! Here's a step-by-step guide:

1. **Create an IAM role**: You'll need an IAM role that allows VPC Flow Logs to publish logs to CloudWatch Logs. Here's how:
   - Open the IAM console and click "Roles" > "Create Role".
   - Choose "EC2" as the use case and click "Next".
   - Search for and select the "AWSVPCFlowLogsPolicyForCloudWatch" policy, then click "Next".
   - Give your role a friendly name like "VPCFlowLogsRole" and click "Create role".

2. **Select the VPC**: Head over to the VPC console and select the VPC you want to enable Flow Logs for.

3. **Create a Flow Log**: 
   - In the VPC details page, click on the "Flow Logs" tab and then "Create flow log".
   - Choose "Reject" as the filter (this will log only rejected traffic, which is usually more interesting from a security perspective).
   - Select "Send to CloudWatch Logs" as the destination.
   - Choose the IAM role you created earlier.
   - Give your log a fun name and click "Create".

4. **Verify the setup**: After a few minutes, you should start seeing VPC Flow Log events in CloudWatch Logs. You can view them by going to the CloudWatch console, clicking on "Logs" > "Log groups", and selecting the log group that matches your Flow Log name.

That's it! You now have VPC Flow Logs enabled and ready to help you spot any fishy business in your network.

## What are some gotchas?
While setting up VPC Flow Logs is pretty straightforward, there are a few things that might trip you up:

1. **IAM permissions**: Make sure the IAM role you use for VPC Flow Logs has the necessary permissions to publish logs to CloudWatch. Specifically, it needs the `logs:CreateLogGroup`, `logs:CreateLogStream`, and `logs:PutLogEvents` permissions for the CloudWatch Logs resource. If you use the managed "AWSVPCFlowLogsPolicyForCloudWatch" policy, you should be good to go.

2. **Log group name collision**: If you have multiple VPCs and use the same log group name for their Flow Logs, the logs will be mixed together, which can make analysis more difficult. To avoid this, use unique log group names for each VPC, or better yet, use a consistent naming convention that includes the VPC ID.

3. **Log retention**: By default, CloudWatch Logs will keep your VPC