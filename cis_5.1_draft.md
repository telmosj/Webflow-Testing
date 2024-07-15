# 5.1 Ensure no Network ACLs allow ingress from 0.0.0.0/0 to remote server administration ports (Automated)

## Summary
Hey there! Let's chat about an important security recommendation from the fine folks at the Center for Internet Security (CIS). They suggest making sure your Network Access Control Lists (NACLs) in AWS don't allow unrestricted inbound access from any IP address to remote admin ports like SSH and RDP. Tightening this up can help prevent attackers from accessing your servers.

## Where did this come from?
This stellar security advice comes straight from the CIS Amazon Web Services Foundations Benchmark v3.0.0 - 01-31-2024. You can download the full benchmark chock-full of other useful recommendations at https://downloads.cisecurity.org/#/. The friendly AWS documentation also provides helpful guidance on NACLs and VPC security that complements this CIS recommendation.

## Who should care? 
This matters for:
- Cloud engineers responsible for securing AWS environments
- DevOps teams deploying servers and applications in AWS 
- Security analysts assessing the security posture of AWS accounts
- Compliance officers ensuring adherence to security best practices

## What is the risk?
Leaving NACLs wide open to allow unfettered access to sensitive admin ports from anywhere on the internet is asking for trouble. It greatly expands the attack surface, making it easy for hackers to discover and brute force their way into your servers. Compromised servers could lead to data breaches, malware infections, and systems being comandeered for nefarious purposes. Following this CIS recommendation goes a long way in making remote server access much harder for the bad guys.

## What's the care factor?
On a scale from "meh" to "mega important", this one rates pretty high. Overly permissive NACLs are a common security gap that attackers actively seek out. Considering the huge downside risk of a breach versus the relatively low effort to tighten NACLs, this is absolutely worth prioritizing and getting right. Don't snooze on this one!

## When is it relevant?
Carefully restricting NACL inbound rules makes sense anytime you have EC2 instances, containers, or other server resources deployed in a VPC. It's especially critical for production environments and servers housing sensitive data. 

The rare cases where wide open NACLs might be okay is in isolated dev/test environments with no sensitive data and limited blast radius if compromised. But as a general rule, following this CIS recommendation is almost always the way to go.

## What are the trade-offs?
Tightening NACLs does require some extra configuration effort upfront. You'll need to determine the specific IP ranges of your admins and other authorized users that require access. Overly restrictive rules could temporarily break legitimate access until properly tuned.

However, this pales in comparison to the major security benefits. A few minutes spent upfront can avoid a major incident and all the time, cost and reputation damage that comes with it. The security juice is definitely worth the squeeze here.

## How to make it happen?
Time to walk through implementing this CIS recommendation in your AWS environment:

1. Sign into the AWS Management Console and open the VPC dashboard 
2. On the left menu, click "Network ACLs"
3. Select the NACL you want to secure
4. Go to the "Inbound Rules" tab
5. Check if there are any rules with:
   - Port range including 22 (SSH), 3389 (RDP), or other admin ports
   - Protocols TCP, UDP or ALL 
   - Source IP of 0.0.0.0/0 (anywhere)
   - ALLOW effect
6. If any such rules exist, either:
   - Click "Edit inbound rules" and modify the Source to be a narrower IP range of authorized admins 
   - Or just delete the risky rule altogether by clicking "Delete"
7. Click "Save" to apply the changes
8. Rinse and repeat this process for all other NACLs in the VPC

## What are some gotchas?
When locking down NACLs, make sure you have the `ec2:DescribeNetworkAcls` IAM permission. Without it, you won't even be able to view the NACL rules.

To actually make changes, you'll need additional permissions like `ec2:CreateNetworkAclEntry`, `ec2:DeleteNetworkAclEntry`, and `ec2:ReplaceNetworkAclEntry` as described in the [AWS docs](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-policy-examples.html). 

Be extra cautious not to accidentally lock yourself out of the servers in the process! Double check your rules before saving.

Keep in mind NACLs are stateless, so you'll need to create both inbound and outbound rules. Don't forget the ephemeral port ranges for outbound.

## What are the alternatives?
NACLs aren't the only way to control access to servers. Some other options to consider:
- Using Security Groups which act as instance-level firewalls. They provide more granular control but are scoped to EC2 instances.
- Implementing a bastion host or jump server as the sole point of entry into the VPC
- Setting up a virtual private network (VPN) for remote access vs exposing admin ports directly
- Employing Zero Trust solutions for more sophisticated access policies

## Explore further
- Check out the related CIS recommendation 5.2 which suggests a similar hardening for outbound ports
- For more on the differences between Security Groups and NACLs, see the [AWS security docs](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Security.html#VPC_Security_Comparison)
- Dive deeper with the official CIS AWS Foundations Benchmark available at https://downloads.cisecurity.org/#/

So there you have it! Go forth and secure those NACLs to keep your cloud castles safe from the ravages of marauding hackers. Your servers will thank you. Stay secure out there!