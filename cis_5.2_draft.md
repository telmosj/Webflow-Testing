# 5.2 Ensure no security groups allow ingress from 0.0.0.0/0 to remote server administration ports (Automated)

## Summary
As a best practice, avoid configuring any security groups in AWS to allow unrestricted inbound access from any IP address (0.0.0.0/0) to remote administration ports like SSH (22) and RDP (3389). Allowing public internet access to these ports significantly increases the attack surface and risk of your EC2 instances getting compromised. Instead, limit inbound access to only the specific IP ranges that require it.

## Where did this come from?
This recommendation comes from the CIS Amazon Web Services Foundations Benchmark v3.0.0 - 01-31-2024. You can [download the full CIS AWS Benchmark](https://downloads.cisecurity.org/#/) for more details and related recommendations. The [official AWS documentation on security groups](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-security-groups.html) is also a great reference.

## Who should care? 
- Cloud Security Engineers responsible for defining secure AWS architectures
- DevOps Engineers configuring VPCs, subnets and security groups 
- IT Security Managers overseeing cloud infrastructure risk
- Penetration Testers assessing EC2 instance security posture
- Compliance Officers enforcing security standards and best practices

## What is the risk?
Allowing inbound access from any IP (0.0.0.0/0) to remote admin ports (22, 3389, etc.) makes it trivial for attackers to discover and brute force their way into your EC2 instances. This could allow them to:

- Gain unauthorized access to sensitive data and systems
- Install malware, crypto miners, or other malicious software 
- Use your resources to attack others or mask their identity
- Cause downtime, data loss, or reputational damage

While not a guarantee, following this recommendation significantly reduces the likelihood and impact of these threats.

## What's the care factor?
This is a critical security recommendation that should be followed in nearly all cases. The risk of leaving RDP/SSH open to the world far outweighs any potential convenience. Cloud security is a shared responsibility - AWS secures the underlying infrastructure, but customers are responsible for securing their EC2 instances. Failure to restrict access to remote admin ports is an easily avoidable but extremely common EC2 configuration mistake that attackers actively scan for.

## When is it relevant?
This recommendation applies to any EC2 instance or other resource (RDS, ElastiCache, etc.) placed in a VPC and assigned a security group. It's especially important for:

- Internet-facing instances or those in public subnets
- Instances holding sensitive data (PII, financial, IP, etc.)  
- Mission-critical production workloads

Situations where it may not apply:

- Instances in isolated private subnets not reachable from the internet
- Instances used for testing or research that hold no sensitive data
- Instances quickly launched and torn down (still recommended if possible)

## What are the trade-offs?
Restricting inbound access to remote admin ports does require some additional effort versus leaving them completely open. Potential issues to consider:

- Need to maintain a list of allowed IPs/ranges that require admin access
- May require a bastion host or VPN to first access the private network
- Slightly less convenient for admins who travel or work from home 
- May break existing tools that assume unrestricted SSH/RDP access

However, for most organizations, the security benefits far outweigh these challenges. Many can be solved with proper network design (VPNs, bastions) and updated admin processes.

## How to make it happen?
To check if any security groups have unrestricted inbound rules for remote admin ports:

1. Open the [AWS Management Console](https://console.aws.amazon.com/vpc/) and navigate to the VPC Dashboard
2. Click "Security Groups" in the left sidebar 
3. Select a security group from the list
4. Switch to the "Inbound rules" tab
5. Look for any rules that have the following:
   - Port range including 22 (SSH), 3389 (RDP), or other remote admin ports
   - Protocol of TCP, UDP or ALL 
   - Source of 0.0.0.0/0 (any IP) or ::/0 (any IPv6)
6. Repeat for other security groups

To remediate:

1. Select the offending security group
2. In the "Inbound rules" tab, click "Edit inbound rules"  
3. Click the "x" to delete the overly permissive rules 
   - Alternatively, modify the "Source" to be a specific IP or range
4. Click "Save rules"
5. Repeat the process for other impacted security groups

Remember, you can always grant inbound SSH/RDP access to your specific IP as needed. But the default state of security groups should be deny all.

## What are some gotchas?
- Deleting a permissive inbound rule may disrupt existing admins' access to EC2 instances. Communicate the change in advance.
- The EC2 launch wizard allows choosing a "default" security group that allows inbound SSH from anywhere. Always customize this.
- Avoid using the "ALL" protocol or port range unless absolutely required. Be as specific as possible.
- Don't just set Source to 0.0.0.0/0 for convenience. Allowed IPs should be a conscious decision.
- Modifying a security group's rules affects all EC2 instances associated with it. Double check before making changes.

Also remember that to make any changes to security group rules, you need the `ec2:AuthorizeSecurityGroupIngress` and `ec2:RevokeSecurityGroupIngress` permissions. See the [official docs](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_AuthorizeSecurityGroupIngress.html) for details.

## What are the alternatives?
Other ways to reduce the risk of open remote admin ports:

- Use a bastion host in a public subnet as the only ingress point for SSH/RDP
- Establish a VPN connection from your office/home to access private EC2 instances
- Only launch EC2 instances in private subnets with no internet gateway
- Use AWS Systems Manager Session Manager for remote shell access without SSH  

However, restricting inbound access at the security group level is still recommended as a foundational best practice.

## Explore Further 
- CIS AWS Foundations Benchmark Recommendation 5.3 for limiting outbound traffic from security groups
- CIS AWS Foundations Benchmark Recommendation 5.1 for avoiding the default VPC
- [AWS Security Best Practices whitepaper](https://aws.amazon.com/architecture/security-identity-compliance/)
- [How to Bastion host](https://aws.amazon.com/blogs/security/how-to-record-ssh-sessions-established-through-a-bastion-host/)