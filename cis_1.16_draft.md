# 1.16 Ensure IAM policies that allow full "*:*" administrative privileges are not attached (Automated)

## Summary
It's easy to get carried away when creating IAM policies and granting more permissions than are really needed. However, following the principle of least privilege is a security best practice for good reasons. Overly permissive policies, especially ones that allow full "*:*" administrative privileges, can be a major security risk if attached to users, groups, or roles.

## Where did this come from?
This recommendation comes from the CIS Amazon Web Services Foundations Benchmark v1.4.0 - 05-23-2022. You can download the full CIS benchmark from https://downloads.cisecurity.org/#/. 

The CIS AWS Foundations Benchmark provides prescriptive guidance for configuring security options for a subset of Amazon Web Services with an emphasis on foundational, testable, and architecture-agnostic settings. Recommendation 1.16 falls under the Identity and Access Management category.

For more information and guidance from AWS, see:
- [IAM Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html) 
- [Overview of Access Management: Permissions and Policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction_access-management.html)
- [Security Best Practices in IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html#grant-least-privilege)

## Who should care?
This is most relevant for:
- Cloud/DevOps engineers responsible for designing and implementing IAM policies
- Security engineers assessing the security posture of AWS accounts
- Compliance officers ensuring adherence to security standards and avoiding audit findings
- Developers using AWS services that need to follow organizational security policies

## What is the risk?  
IAM policies that allow full "*:*" administrative privileges provide unrestricted access to all actions on all resources in an AWS account. This means any user, group, or role with this policy attached can perform any action on any resource, essentially making them an account admin.

Risks of overly permissive policies like this include:
- Accidental deletion or modification of critical resources 
- Easier path for attackers to cause damage if they compromise a user/group/role
- Difficulty enforcing separation of duties
- Audit findings and non-compliance with security standards

While policies with full admin rights can be convenient, the risks outweigh the benefits in most cases. The blast radius is just too large if something goes wrong.

## What's the care factor?
Many security-conscious organizations put a high priority on the principle of least privilege. Avoiding policies that allow full administrative access is a big part of that. 

Cloud/DevOps teams should care a lot about avoiding admin policies, even if it makes IAM more complex. A mature, well-secured AWS environment should have no full-admin IAM policies.

But this takes extra time and effort to design granular policies, test them, and refine. So it's understandable teams that are moving fast may be tempted to use admin policies in non-production environments. Using `AdministratorAccess` for a machine user is an anti-pattern you see often.

The care factor really depends on your risk tolerance, security requirements, and the sensitivity of resources. Production AWS accounts, especially ones handling regulated data, should have a very high standard. Companies that have adopted a cloud operating model with automated policy enforcement also tend to care a lot.

But for startups iterating quickly and running lean, avoiding admin policies may be seen as a "nice to have" versus a "must have". Though it's always good to bake in security from the start as much as possible rather than bolt it on later.

## When is it relevant?
Some examples of when this CIS recommendation is highly relevant:
- Production AWS accounts 
- Environments subject to security audits or compliance requirements 
- AWS accounts with sensitive data or mission critical infrastructure
- Organizations with a multi-account AWS environment that use IAM for access management

Some examples where it may be less relevant:
- Personal AWS training/demo accounts
- Temporary development/test environments that don't have sensitive data and are isolated
- Organizations using an external identity provider for AWS access versus using native IAM  

But in general, it's a good practice and relevant in most scenarios. It's always better to start with minimal permissions and add more as needed vs having to remove excessive permissions later.

## What are the trade-offs?
Implementing least privilege and avoiding full admin policies does have some costs:
- Takes more time to design granular policies and test them iteratively
- Increases the number of policies to manage over time
- Can slow down development velocity if engineers have to keep requesting new permissions
- Frustrating user experience if permissions are too restrictive
- Potential for things to break if policies are misconfigured or incomplete

So there is additional overhead involved. It's not a zero-cost recommendation. But the security benefits are substantial too. It's about striking the right balance and making sure security doesn't become a blocker.

Some ways to make it easier:
- Using IAM Access Analyzer to help refine permissions 
- Leveraging IAM policy conditions for more granular control
- Implementing processes to grant temporary elevated access when needed
- Automating IAM policy management with infrastructure-as-code
- Providing self-service portals for permissions management

So, while it takes extra effort, there are ways to make it smoother. It's an area where investing in automation and tooling can have big payoffs.

## How to make it happen?
Here are the high-level steps to implement this CIS recommendation:

1. Identify any IAM policies that allow full "*:*" administrative privileges. You can do this by:
   - Using the IAM console and reviewing policies
   - Running the AWS CLI command to list policies:
      `aws iam list-policies --only-attached --output text`
   - Using a tool like CloudTrail or AWS Config to assess IAM policies
   
2. For each policy identified, determine what users, groups and roles have that policy attached. The key AWS CLI commands are:
   - List entities for a policy: 
     `aws iam list-entities-for-policy --policy-arn <policy_arn>`
   - List attached user policies:
     `aws iam list-attached-user-policies --user-name <username>`
   - List attached group policies:
     `aws iam list-attached-group-policies --group-name <groupname>`
   - List attached role policies:
     `aws iam list-attached-role-policies --role-name <rolename>`

3. Detach the "*:*" admin policies from all the identified users, groups and roles. The AWS CLI commands are:
   - Detach user policy:
     `aws iam detach-user-policy --user-name <username> --policy-arn <policy_arn>`
   - Detach group policy:
     `aws iam detach-group-policy --group-name <groupname> --policy-arn <policy_arn>`
   - Detach role policy: 
     `aws iam detach-role-policy --role-name <rolename> --policy-arn <policy_arn>`

4. Delete the now detached "*:*" admin policies. In the IAM console:
   - Go to the policy detail page
   - Select "Delete" from the Policy actions dropdown
   - Confirm the deletion

   Or use this AWS CLI command:
   `aws iam delete-policy --policy-arn <policy_arn>`

5. Create new IAM policies that follow the least privilege by only allowing the specific permissions needed. Some tips:
   - Be as granular as possible in terms of allowed actions
   - Restrict permissions to specific resources where feasible 
   - Use IAM policy conditions to further restrict permissions
   - Consider breaking up policies per service or job function
   - Test policies and refine them based on actual access needs

6. Attach the new least privilege policies to the relevant users, groups, and roles.

7. Enable IAM Access Analyzer to continuously monitor permissions and suggest refinements.

So in summary - find overly-permissive policies, detach and delete them, replace with the least privilege policies, and use IAM Access Analyzer to maintain least privilege over time.

## What are some gotchas?
Some things to watch out for when implementing this:
- Deleting an IAM policy that is still attached. You have to detach it from all entities first.
- Inline IAM policies. The recommendation focuses on managed policies but inline policies can also allow full admin access. Be sure to check for those too.
- Resource-level permissions. You can still allow fairly wide access without using "*:*". Be careful of any policy that allows all actions ("*") on a specific resource or uses broad resource definitions.
- Service