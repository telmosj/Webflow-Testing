# 1.22 Ensure access to AWSCloudShellFullAccess is restricted (Manual)

## Summary
Hey there! Let's chat about an important AWS security recommendation from the fine folks at the Center for Internet Security (CIS). They suggest restricting access to the `AWSCloudShellFullAccess` IAM policy. This policy gives users full reign over the AWS CloudShell environment, which could potentially be abused for nefarious purposes like data exfiltration. Better to play it safe and limit who can use it!

## Where did this come from?
This recommendation comes straight from the "CIS Amazon Web Services Foundations Benchmark v3.0.0 - 01-31-2024". You can download the full benchmark document [here](https://downloads.cisecurity.org/#/). The benchmark provides a comprehensive set of security configuration best practices for hardening your AWS environment. For more details on CloudShell security, check out the [AWS CloudShell documentation](https://docs.aws.amazon.com/cloudshell/latest/userguide/sec-auth-with-identities.html).

## Who should care?
This is most relevant for:
- AWS administrators responsible for securing the AWS environment 
- Security engineers tasked with minimizing risks of data loss
- Compliance officers ensuring adherence to security benchmarks and standards

## What is the risk?
The main risk is that a malicious user with full CloudShell access could potentially use it to move sensitive data outside of the AWS environment. Since CloudShell allows file uploads/downloads and internet access, an attacker could install file transfer utilities and exfiltrate data to an external server. They have sudo permissions in the shell, so they can do some real damage if left unchecked.

## What's the care factor?
For most organizations, I'd rate this as a "Better Safe Than Sorry" type of recommendation. While the risk of rogue admins siphoning data out via CloudShell may not be at the top of your threat model, the consequences could be severe. Proactively restricting access is fairly simple, so might as well check this box and sleep a bit sounder. Environments with highly sensitive data should definitely prioritize this more.

## When is it relevant?
Restricting CloudShell access makes sense for:
- Multi-user AWS environments 
- Organizations subject to strict data privacy regulations
- Companies with intellectual property or sensitive data in AWS

It may be less of a concern for:
- Single-user AWS accounts
- Non-sensitive dev/test environments
- Organizations already using other tools for interactive CLI access

## What are the trade-offs?
The main downside of restricting CloudShell access is administrative overhead. You'll need to set up and maintain more granular IAM policies rather than just using the default `AWSCloudShellFullAccess` policy. This means a bit more work for your IAM admins. Users may also be slightly inconvenienced if they were used to the flexible file transfer functionality that the default policy allows. But overall, enhancing your security posture is likely worth these minor trade-offs.

## How to make it happen?
Here's how to check and remediate CloudShell access:

From the Console:
1. Open the IAM console: https://console.aws.amazon.com/iam/  
2. Click on "Policies" in the left sidebar
3. Search for "AWSCloudShellFullAccess"
4. Select the policy and go to the "Entities attached" tab
5. If there are any roles/users/groups attached, check the boxes and click "Detach"

From the CLI:
1. List IAM policies and note the ARN for "AWSCloudShellFullAccess":
```
aws iam list-policies --query "Policies[?PolicyName == 'AWSCloudShellFullAccess']"
```
2. Check if the policy is attached to any roles:
```  
aws iam list-entities-for-policy --policy-arn arn:aws:iam::aws:policy/AWSCloudShellFullAccess
```
3. Ensure `PolicyRoles` is empty `[]` in the output
4. If not empty, detach the policy from the listed roles

## What are some gotchas?
To detach the CloudShell policy, you'll need `iam:DetachRolePolicy` permissions (for role attachments) or `iam:DetachUserPolicy` (for user attachments). The AWS-managed `IAMFullAccess` policy can work, or a more tailored custom IAM policy. See the [IAM API Permissions Reference](https://docs.aws.amazon.com/service-authorization/latest/reference/list_identityandaccessmanagement.html) for details.

Also, keep in mind that other IAM policies attached to users/roles may still allow CloudShell access even if `AWSCloudShellFullAccess` is detached. Be sure to audit all attached policies for `cloudshell:*` permissions.

## What are the alternatives?
If you still need interactive CLI access to AWS, consider:
- Using the [AWS CLI](https://aws.amazon.com/cli/) from local workstations instead
- Deploying your own bastion host with limited permissions 
- Leveraging other managed services like [AWS Systems Manager Session Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager.html) for shell access to EC2 instances

These approaches give you more control and can provide secure access without the full power of the CloudShell environment.  

## Explore Further
For more details on locking down CloudShell, see:
- [Securing Access to Cloudshell with IAM](https://docs.aws.amazon.com/cloudshell/latest/userguide/sec-auth-with-identities.html)
- [AWS IAM Security Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html) 

This guidance aligns with:
- CIS Control v8 6 - Access Control Management
- CIS Control v7 14 - Controlled Access Based on the Need to Know

Stay vigilant and keep those shells in check! Let me know if you have any other questions.