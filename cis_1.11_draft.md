# 1.11 Do not setup access keys during initial user setup for all IAM users that have a console password (Manual)

## Summary
When creating a new IAM user in AWS, it's best practice not to set up their access keys right away, even if you know they will need programmatic access. Instead, make access key creation a separate step that the user must perform themselves later. This provides a clear indication of intent that the keys are actually needed and being used.

## Where did this come from?
This recommendation comes from the CIS Amazon Web Services Foundations Benchmark v3.0.0 - 01-31-2024, which can be downloaded from the [CIS website](https://downloads.cisecurity.org/#/). The benchmark provides prescriptive guidance for configuring AWS accounts with security best practices. Additional details can be found in the [AWS IAM documentation](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html#create-iam-users).

## Who should care? 
This recommendation is relevant for:
- IAM administrators responsible for creating and managing IAM users
- Security teams who want to ensure IAM best practices are being followed 
- Compliance officers validating that access is granted according to the principle of least privilege

## What is the risk?
Setting up access keys during initial user creation can lead to:
- Unnecessary access being granted, violating least privilege 
- Increased attack surface if keys are created but not actually needed
- Lack of visibility into which users actually require and are using programmatic access

While this practice won't directly lead to a security incident on its own, it contributes to loose access management that attackers could abuse.

## What's the care factor?
For most organizations, avoiding this practice should be considered a low to medium priority. It's a relatively easy change to make, and enforces good IAM hygiene, but isn't an acute risk. Focus on this after implementing more fundamental IAM best practices like enforcing MFA, rotating keys regularly, and removing inactive users.

## When is it relevant?
This recommendation is relevant for all new IAM users being created, regardless of job function. The only exception would be service accounts that must be created with programmatic access and don't have a real person associated with the account.

## What are the trade-offs?
The main downside to this approach is a slightly longer process for setting up new users. Administrators will need to communicate to users how to set up their own access keys as a separate step. Users may find this inconvenient. However, this is outweighed by the security benefits of avoiding unnecessary access grants.

## How to make it happen?
To properly set up a new IAM user without access keys:

1. In the IAM console, select "Add user"
2. Enter a username and check the box for "Password - AWS Management Console access"
3. Choose whether to auto-generate the password or create a custom one 
4. Leave the checkbox for "Access key - Programmatic access" unchecked
5. Click through to complete user creation without adding any permissions yet
6. Send the new user their console sign-in URL and initial password
7. Direct the user to sign in and create their own access keys if needed following [these steps](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html#Using_CreateAccessKey)

## What are some gotchas?
- Remember that users will need the `iam:CreateAccessKey` permission to create their own access keys. Make sure your IAM policies grant this.
- If you use AWS Organizations with Service Control Policies, verify that these don't restrict the ability for users to self-manage their access keys.
- Access keys can only be created for one IAM user at a time. You cannot bulk create them for multiple users at once.

## What are the alternatives?
Instead of having users create their own keys, an administrator could create keys as a separate step after initial user creation. However, this still requires the extra step. It may also be harder to track which users have been provisioned keys. Having users self-serve ensures each key creation is logged under their own name.

## Explore further
- Brush up on other IAM best practices in the [CIS AWS Foundations Benchmark](https://downloads.cisecurity.org/#/) 
- Walk through the IAM tutorial in the [AWS documentation](https://docs.aws.amazon.com/IAM/latest/UserGuide/tutorial_security-creds-console.html)
- Learn more about the principle of least privilege access in the [AWS Well-Architected Framework Security Pillar](https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/identity-and-access-management.html)