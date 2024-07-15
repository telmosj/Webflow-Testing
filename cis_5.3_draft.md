# 5.3 Ensure no security groups allow ingress from ::/0 to remote server administration ports (Automated)

## Summary

Hey there! Let's chat about a really important security recommendation from the clever folks at the Center for Internet Security (CIS). They've got some great advice about properly configuring your AWS security groups. The key takeaway is to make sure you're not accidentally leaving remote server administration ports, like SSH and RDP, wide open to the entire internet. Trust me, you don't want that!

## Where did this come from?

This sage advice comes straight from the "CIS Amazon Web Services Foundations Benchmark v3.0.0 - 01-31-2024". You can download the full benchmark document from the CIS website at https://downloads.cisecurity.org/#/. It's chock-full of other great security recommendations for keeping your AWS environment locked down tight.

For more details on working with security groups in AWS, check out the official AWS documentation: 
- [Amazon EC2 Security Groups](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-security-groups.html)
- [Security Groups for Your VPC](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html)

## Who should care?

This is essential knowledge for:
- Cloud Security Engineers with responsibility for defining AWS network security policies
- DevOps Engineers with a need to ensure their AWS environments are properly secured
- Solutions Architects who want to bake security best practices into their AWS designs from the start

## What is the risk?  

Leaving remote administration ports exposed to the entire internet (::/0 in IPv6 notation) is like leaving your front door wide open with a big flashing neon sign that says "Hack me!". It greatly increases the attack surface of your AWS resources and makes it far too easy for bad actors to compromise them. 

Exposed SSH (port 22) and RDP (port 3389) ports could allow attackers to brute force their way in and gain unauthorized access to your EC2 instances. From there, they could steal sensitive data, install malware, use your resources for illicit activities, or pivot to attack other systems. It's bad news all around.

Properly restricting access to these ports with security groups is an effective way to mitigate these risks. It won't stop every determined attacker, but it puts a big obstacle in their path.

## What's the care factor?

For anyone responsible for the security of AWS environments, this should be a high priority item to address ASAP. The consequences of a breach through this vector could be severe - data loss, reputational damage, financial penalties, operational disruption. It's really not something you want to learn the hard way.

Even if you think your EC2 instances aren't an attractive target or don't contain sensitive data, leaving them exposed is still poor hygiene. Attackers can use them as part of botnets or for crypto mining. You could rack up an unexpectedly large AWS bill or get your account suspended. 

So in short, you should care a lot! Locking down remote admin ports with security groups should be standard practice for every AWS environment.

## When is it relevant?

This recommendation is relevant for pretty much any AWS environment that uses EC2 instances and security groups. That covers a lot of ground!

Some specific examples where it's especially important:
- Internet-facing web servers and API endpoints 
- Bastion hosts used to administer other systems
- Instances running databases, caches, message queues etc that shouldn't be directly accessible
- Dev/test environments that were quickly spun up without following best practices

There are a few scenarios where it may be less applicable:
- Instances that don't have SSH/RDP enabled at all (fairly rare)
- Environments that use a different mechanism to restrict admin access, like AWS Systems Manager Session Manager
- Non-production environments that are very short-lived and isolated (but even then, better safe than sorry!)

## What are the trade offs?

The main potential downside of implementing this recommendation is making remote administration a bit less convenient. If you completely remove the inbound rules for SSH/RDP from security groups, you won't be able to directly connect to those ports from anywhere. 

That means you may need to set up a bastion host or VPN connection to first get inside the VPC, then connect to the instances. Or use an alternative like Systems Manager Session Manager. This adds some complexity and an extra hop.

You could also allow SSH/RDP from specific whitelisted IP ranges rather than completely removing access. But that can be brittle and tricky to maintain, especially if admins' IPs change often.

In the grand scheme though, this is a very worthwhile trade off for the significant security benefits. The small inconvenience is far outweighed by the reduced risk.

## How to make it happen?

Here's how to check for and remove any inbound rules for remote admin ports that are open to ::/0  (IPv6 any source) in your security groups:

1. Open the AWS Management Console and navigate to the VPC service 
2. In the left sidebar, click on "Security Groups"
3. Select a security group from the list
4. Go to the "Inbound rules" tab
5. Look for any rules that have ::/0 as the source and include port 22 (SSH), 3389 (RDP), or any other remote admin ports you use
   - This could be a single port, a range like 0-1024, or "All"
6. If you find any, click the "Edit inbound rules" button 
7. Click the "Delete" button next to each offending rule to remove it entirely
   - Alternatively, you can change the "Source" to be a specific IP range rather than ::/0 if you still need some access
8. Click "Save rules" to apply the changes
9. Repeat this process for all security groups in all VPCs and regions

Tip: You can also use the AWS CLI to query for bad rules across all security groups and pass the output to another command to remove them. Here's an example:
aws ec2 describe-security-groups --query 'SecurityGroups[?length(IpPermissions[?FromPort==`22` && contains(IpRanges[].CidrIp, `::/0`)]) > `0`].{GroupId: GroupId}' --output text | xargs aws ec2 revoke-security-group-ingress --group-id {} --protocol tcp --port 22 --cidr ::/0

## What are some gotchas?

A few things to watch out for when tightening up your security groups:
- Make sure you have an alternative way to access the instances for administration before removing the inbound rules. Otherwise you could lock yourself out!
  - Consider setting up a bastion host, VPN, AWS Systems Manager Session Manager, etc first
- Removing rules is a destructive action. Make sure you really want to fully remove access vs just limiting the source IPs.
- Don't forget to check all security groups in all regions and VPCs. It's easy to miss some when you have a lot of them.
- The security group changes take effect immediately. There's no way to roll back. So tread carefully, especially in production!
- To apply the rules, you need permissions to call at least:
  - ec2:DescribeSecurityGroups
  - ec2:RevokeSecurityGroupIngress 
  - ec2:AuthorizeSecurityGroupIngress (if restricting to specific IPs)
  - See the [Actions, resources, and condition keys for Amazon EC2](https://docs.aws.amazon.com/service-authorization/latest/reference/list_amazonec2.html)

## What are the alternatives?

Some other ways to restrict access to remote admin ports, instead of or in addition to security groups:
- Use [AWS Systems Manager Session Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager.html) to connect to instances via the AWS Console/API without exposing any ports
- Put the instances in a private subnet with no internet gateway, and access them through a bastion host in a public subnet
- Use a VPN connection from your office/home network to the VPC
- Implement [Zero Trust network access](https://aws.amazon.com/security/zero-trust/) principles to restrict access to the minimum required

## Explore further

Some additional resources to learn more about AWS Security Groups and securing remote access:
- [AWS Security Group Docs](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-security-groups.html) 
- [AWS Guide to Network Security](https://docs.aws.amazon.com/whitepapers/latest/aws-security-whitepaper/network-security.html)