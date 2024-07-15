# 1.21 Ensure IAM users are managed centrally via identity federation or AWS Organizations for multi-account environments (Manual)

## Summary
If you're using multiple AWS accounts, it's important to manage your IAM users centrally. You can do this by either federating with an external identity provider or using AWS Organizations. This reduces complexity and makes it less likely you'll make access management mistakes.

## Where did this come from?
This recommendation comes from the CIS Amazon Web Services Foundations Benchmark v3.0.0 - 01-31-2024. You can download the full benchmark from the [CIS website](https://downloads.cisecurity.org/#/). The benchmark provides prescriptive guidance for configuring security options for a subset of Amazon Web Services with an emphasis on foundational, testable, and architecture agnostic settings.

For more background, check out the [AWS IAM documentation on identity providers and federation](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_providers.html) and the [AWS Organizations User Guide](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_introduction.html).

## Who should care? 
This is most relevant for:
- IAM administrators responsible for managing user access across multiple AWS accounts
- Security engineers looking to reduce the risk of unauthorized access in multi-account environments
- Compliance officers ensuring adherence to security benchmarks and best practices

## What is the risk?
When you have IAM users spread across multiple AWS accounts, it becomes much more difficult to manage their access in a consistent and secure manner. This can lead to:

- Users accumulating excessive permissions over time as they move between accounts and roles
- Stale user accounts that are no longer needed but still have active credentials
- Lack of visibility into who has access to what across the entire organization
- Increased likelihood of misconfigurations and access control mistakes

Centralizing IAM user management significantly reduces these risks by providing a single place to control and audit access. While it can't eliminate all access risks, it's a foundational step in establishing secure identity management practices.

## What's the care factor?
For most organizations operating in AWS, centralizing IAM should be a high priority initiative. The downside risks of poorly managed decentralized access are simply too high - a single compromised user account or overly permissive role can lead to large-scale data breaches, destructive attacks, or financial losses.

Centralizing IAM forces you to be proactive and disciplined about managing access. It requires some upfront work, but pays off immensely in terms of enhanced security, simplified administration, and easier compliance. I would strongly recommend tackling this sooner rather than later.

## When is it relevant?
Centralized IAM management is relevant for:

- Any organization using multiple AWS accounts, even if it's just a few
- Organizations subject to compliance regimes that require strict access controls and auditing
- Businesses with more than a handful of employees accessing AWS resources

It may be overkill for:

- Very small deployments using a single AWS account 
- Workloads entirely managed by a single administrator
- Test or dev environments that are isolated from production

When in doubt, it's better to err on the side of centralizing IAM. You can always partition access further within a centralized model as needed.

## What are the trade-offs?
Centralizing IAM requires some discipline and changes to how you manage access across accounts. Some of the key considerations are:

- Upfront effort to design and implement the centralized model (SSO, AWS Organizations, etc.)
- Getting buy-in from account owners and administrators on the new model
- Potential disruptions as you migrate existing IAM users and resources over
- Ongoing overhead of managing centralized access policies and provisioning
- Some added complexity for end users having to navigate SSO login flows

While these are real trade-offs, I would argue they are vastly outweighed by the security benefits in most cases. A well-implemented central IAM model can actually *decrease* overhead and complexity once in place.

## How to make it happen?
The exact steps will depend on your chosen centralized IAM approach, but at a high level:

1. Designate a central IAM account that will manage all users and access policies. This could be a dedicated account or your AWS Organizations master account.

2. Connect your central IAM account to an external identity provider (if using SSO) or AWS Organizations (if using native AWS centralization). 

3. In the central IAM account, create IAM roles that grant access to your other AWS accounts. Use IAM policies to define the specific permissions for each role.

4. Create IAM policies in the central account that allow users to assume those cross-account roles. Attach the policies to your IAM users or groups.

5. Modify the trust relationships of the cross-account roles to allow the central account's IAM users to assume them.

6. Remove any existing IAM users from your other AWS accounts, so that all access is managed through the central account.

7. Test and verify the end-to-end SSO or cross-account access flow for your users.

8. Communicate the new access model to your users and provide them with updated instructions.

For detailed steps and best practices, see the [AWS IAM Identity Center documentation](https://docs.aws.amazon.com/singlesignon/latest/userguide/what-is.html) and the [AWS Organizations Access Management Guide](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_accounts_access.html).

## What are some gotchas?
Some things to watch out for when implementing centralized IAM:

- Beware of any hard-coded access keys or IAM user credentials in application code or configuration files, as these will need to be updated. Use IAM roles instead where possible.

- Ensure your IAM policies follow least privilege and only grant the minimum permissions necessary for each user's job function. Overly broad policies can defeat the purpose of centralization.

- When connecting to an external identity provider, pay close attention to the IAM trust policy to ensure only authorized users and roles can assume the AWS roles.

- The IAM permissions boundary feature can help enforce guardrails on cross-account access. Use it to set maximum permission limits that apply to all IAM entities.

- Throughout the process, you'll need IAM permissions like `iam:CreateRole`, `iam:CreatePolicy`, `sts:AssumeRole`, `organizations:DescribeAccount` and others. Check the [Actions, Resources, and Condition Keys pages](https://docs.aws.amazon.com/service-authorization/latest/reference/reference_policies_actions-resources-contextkeys.html) for the specific permissions needed.

## What are the alternatives?
The main alternatives to centralized IAM for multi-account access are:

1. Using separate IAM users and policies in each account, and manually synchronizing them. This is error-prone and scales poorly.

2. Relying entirely on AWS account-level access controls like [consolidated billing](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/consolidated-billing.html). This lacks the granularity of IAM and can lead to over-permissioning.

3. Using third-party access management tools that synchronize IAM across accounts. These can work but add cost and complexity vs. native AWS solutions.

In general, using AWS's built-in identity federation or AWS Organizations capabilities will provide the best balance of security, usability, and native integration.

## Explore further
- [CIS Controls v8](https://www.cisecurity.org/controls/cis-controls-list/) - relevant controls include #5 (Account Management) and #6 (Access Control Management)
- [Amazon IAM Identity Center Best Practices](https://docs.aws.amazon.com/singlesignon/latest/userguide/iam-auth-access-best-practices.html)  
- [AWS Organizations Best Practices](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_best-practices_mgmt-acct.html)
- [Organizing Your AWS Environment Using Multiple Accounts ](https://docs.aws.amazon.com/whitepapers/latest/organizing-your-aws-environment/organizing-your-aws-environment.html)
- Companion AWS Blog Post - [Setting up IAM Identity Center (successor to AWS Single Sign-On)](https://docs.aws.amazon.com/singlesignon/latest/userguide/get-set-up-for-idc.html)