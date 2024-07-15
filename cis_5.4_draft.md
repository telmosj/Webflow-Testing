# 5.4 Ensure the default security group of every VPC restricts all traffic (Automated)

## Summary
Hey there! Let's chat about a super important security recommendation from the smart folks at the Center for Internet Security (CIS). They say that the default security group in your Amazon Virtual Private Cloud (VPC) should be locked down tighter than a drum, blocking all inbound and outbound traffic. This helps keep your cloud resources safe and encourages you to thoughtfully set up custom security groups with only the necessary permissions.

## Where did this come from?
This wisdom comes straight from the CIS Amazon Web Services Foundations Benchmark v3.0.0 - 01-31-2024. You can download the full benchmark document chock-full of other useful recommendations at https://downloads.cisecurity.org/#/. The benchmark provides prescriptive guidance for configuring security options for a subset of Amazon Web Services with an emphasis on foundational, testable, and architecture agnostic settings. 

For more nitty-gritty details on VPC security groups, check out:
- [AWS Security Groups Docs](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html) 
- [AWS Security Group Rules Reference](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/security-group-rules-reference.html)

## Who should care?
This matters for:
- Cloud Security Engineers looking to properly secure AWS environments
- DevOps Engineers interested in implementing security best practices in infrastructure-as-code 
- Solutions Architects designing secure, multi-tiered architectures in VPCs
- IT Managers overseeing teams responsible for cloud security posture

## What is the risk?
Leaving the default security group wide open is like leaving your front door unlocked with a big "rob me" sign. It allows any traffic from any source to reach your cloud resources. This could enable:
- Unauthorized access to your instances and data
- Malware delivery and lateral movement of threats
- Exposure of vulnerable services to the public internet
- Potential privilege escalation and instance compromise

While the default "allow all" rules may be convenient for quick testing or experimentation, they should never be used in production environments. Having a tightly restricted default security group across all VPCs helps prevent accidental exposure and enforces the concept of least privilege.

## What's the care factor?
For anyone with a public-facing cloud deployment, implementing this recommendation should be a high priority. The risk of unintended exposure is amplified in complex environments with dozens or hundreds of instances. A single forgotten openDev instance in the default security group could provide a foothold for attackers.

While private VPCs may be less exposed, the default security group is still the "source of truth" that new instances will use unless explicitly placed in a different group. So it's a good security hygiene practice regardless of your architecture. An ounce of prevention is worth a pound of cure, as they say.

## When is it relevant?
This recommendation is relevant for:
- ALL public-facing VPCs (e.g. web apps, API services) 
- Private VPCs that auto-assign the default security group to new instances
- Organizations subject to compliance mandates like PCI DSS that require tight network controls

It may be less critical for:
- Isolated test/dev environments with no sensitive data
- Fully private, single-purpose VPCs where security groups are always explicitly assigned

But in general, it's a good foundational practice for all VPC deployments.

## What are the trade-offs?
Locking down the default security group does require more upfront planning to ensure your resources are launched with properly scoped security group permissions from the get-go. This can slow down development velocity if security groups are managed manually. 

It also removes the "easy button" of just poking holes in the default group for testing or troubleshooting. Temporary rules have a habit of becoming permanent when left in the default group.

Some additional costs may be incurred for time spent defining and managing custom security groups. But this pales in comparison to the potential cost of a breach.

## How to make it happen?
Here's how to check and remediate your default security groups across all regions:

Security Group State:
1. Login to AWS Management Console and open the VPC dashboard
2. For each VPC (including default):
3. Select the default security group 
4. Click Inbound Rules, ensure no rules exist
5. Click Outbound Rules, ensure no rules exist
6. Remove any existing rules

Security Group Members:  
1. Note the ID of the default security group
2. Open the EC2 Dashboard
3. Filter instances by the default group ID
4. Create new security groups with least privilege rules for these instances 
5. Reassign the instances to the new groups
6. Remove the instances from the default group

Pro tip: tag the default groups with something like "DO NOT USE" after locking them down to make the intent clear.

## What are some gotchas?
- Ensure you have a way to access your instances (e.g. via SSH, SSM) before removing the default group rules
- Be cautious when modifying groups assigned to running instances, it can cause availability issues if not planned carefully
- You need the `ec2:AuthorizeSecurityGroupIngress` and `ec2:AuthorizeSecurityGroupEgress` permissions to modify security group rules

## What are the alternatives?
Instead of (or in addition to) restricting the default groups, you could also:
- Use AWS Firewall Manager to apply security group policies across accounts
- Implement network ACLs as an additional layer of defense at the subnet level
- Use security group references in AWS CloudFormation templates for consistent rulesets

But modifying the default groups is still a best practice, as it reduces the risk of accidental exposure through instance misconfigurations.

## Explore further
- CIS AWS Foundations Benchmark - 
([https://www.cisecurity.org/benchmark/amazon_web_services/](https://www.cisecurity.org/benchmark/amazon_web_services/))
- AWS VPC Security Groups - ([https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html))  
- CIS Controls v8 - ([https://www.cisecurity.org/controls/](https://www.cisecurity.org/controls/))
    - 3.3 Configure Data Access Control Lists
    - 12.3 Securely Manage Network Infrastructure
- CIS Controls v7
    - 14.6 Protect Information through Access Control Lists

This is one of many recommendations in the CIS benchmark focused on securing your AWS environment. Stay tuned for more deep dives on other critical security configurations. Stay safe out there!