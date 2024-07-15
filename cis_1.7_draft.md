# 1.7 Eliminate use of the 'root' user for administrative and daily tasks (Manual)

## Summary
The 'root' user in an AWS account has unrestricted access to all resources and should be used sparingly, if at all. Everyday administrative tasks should be performed with less-privileged IAM users following the principles of least privilege. Monitoring and alerting on 'root' user activity is recommended to detect any unauthorized usage.

## Where did this come from?
This recommendation comes from the CIS Amazon Web Services Foundations Benchmark v3.0.0 - 01-31-2024. You can download a copy of the full benchmark from the [CIS website downloads page](https://downloads.cisecurity.org/#/). The benchmark provides prescriptive guidance for configuring AWS accounts and services to align with security best practices. For more background, refer to the [CIS AWS Foundations Benchmark page](https://www.cisecurity.org/benchmark/amazon_web_services/) and [AWS documentation on IAM best practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html).

## Who should care?
This recommendation is relevant for:
- AWS account administrators responsible for securing the AWS environment 
- Security engineers looking to enforce the principle of least privilege
- Compliance officers ensuring adherence to security standards and frameworks
- Developers and DevOps practitioners performing administrative tasks in AWS

## What is the risk?
The 'root' user has complete unrestricted access to all AWS services and resources in the account. Usage of the root account, especially for daily tasks, introduces several major risks:

- **Accidental misconfiguration or deletion of critical resources** - Even a small mistake when using the 'root' user can have catastrophic consequences due to the unrestricted permissions.

- **Harder to manage and audit activity** - Actions performed by the 'root' user bypass IAM controls making it difficult to implement granular permissions and track activity. 

- **Unnecessary exposure in the event of compromise** - If the root credentials are compromised, an attacker gains immediate access to all resources in the account. This is much worse than a single IAM user being compromised.

By eliminating usage of the 'root' user as much as possible, organizations can significantly reduce the risk of accidental or malicious actions leading to a major incident.

## What's the care factor?
For most organizations, restricting root user access should be considered a high priority security control. The impact of unintended changes or malicious activity performed by the root user is likely to be major, potentially causing significant downtime, data loss, reputational damage or unexpected costs.

Building processes and guardrails to govern the usage of the 'root' user is very achievable and should be one of the first steps taken when configuring a new AWS account. The ongoing effort to maintain these controls is minimal compared to the risks of leaving 'root' usage unchecked.

## When is it relevant?
Restricting 'root' user access is applicable in all AWS accounts, regardless of their purpose or environment. Even in non-production or temporary/disposable accounts, following this recommendation is considered good hygiene and builds the right habits. 

However, there are some specific situations where use of the 'root' user is still required, such as:
- Changing account details like the account name, email address or root user password
- Enabling some high-impact services like CloudTrail or AWS Config for the first time
- Closing the AWS account

A full list of tasks that require the 'root' user can be found in the [AWS documentation](https://docs.aws.amazon.com/general/latest/gr/aws_tasks-that-require-root.html).

## What are the trade-offs? 
The main downside of restricting 'root' user access is the additional overhead required to perform the initial account configuration and distribute credentials for non-root administrative users. 

Some organizations may be tempted to use the 'root' user for speed and simplicity, especially when first starting out with AWS. However, this is a short-sighted approach that prioritizes mild convenience over major risk reduction.

In a large organization with many AWS accounts, the effort of restricting 'root' consistently does require some up-front planning and ongoing diligence. But with the right processes in place, it quickly becomes a standard operating procedure.

## How to make it happen?
1. **Change the 'root' user password to a long, complex value** and store it securely using an enterprise password manager. Avoid using the same password across multiple accounts.

2. **Enable multi-factor authentication (MFA)** for the 'root' user, ideally using a hardware token rather than virtual MFA.

3. **Delete or deactivate any access keys** associated with the 'root' user. Access keys allow programmatic access via the AWS CLI or SDKs and are not needed for regular console login.

4. **Create IAM users** with administrative privileges for yourself and other individuals who require elevated access. Follow the principle of least privilege, only granting the specific permissions required.

5. **Use groups to assign permissions** rather than attaching policies directly to users. This makes it easier to update and manage permissions consistently.

6. **Enforce MFA for all IAM users** to add an extra layer of protection, especially for those with administrative access.

7. **Store IAM user access keys securely** and rotate them regularly. Consider using tools like Hashicorp Vault or AWS Secrets Manager.

8. **Implement a process for using the 'root' user** that requires authorization from security/compliance teams and uses a 'break glass' process for emergency access. 

9. **Monitor and alert on 'root' user activity** using AWS CloudTrail and AWS Config rules to detect any unauthorized access.

## What are some gotchas?
To fully remove the need to use the 'root' user, some specific IAM permissions need to be granted to administrative users:

- `iam:CreateAccountAlias` - Allows renaming the AWS account alias
- `iam:UpdateAccountPasswordPolicy` - Required to update the password policy 
- `iam:CreateServiceLinkedRole` - Used when enabling some services for the first time

If administrative IAM users are missing these permissions, further use of the 'root' user may be needed.

When implementing MFA for the 'root' user and IAM users, virtual MFA should be avoided for the 'root' user and any critical accounts. A lost or wiped phone could lead to a lockout situation. 

Deleting access keys for the 'root' user sounds final, but these can actually be re-created at any time (as the 'root' user has unlimited permissions). Therefore the access key ID should be treated as sensitive even after deletion.

## What are the alternatives?
There is no direct replacement for the 'root' user, as it has a special place in the AWS account structure. However, with careful use of IAM users, groups, roles and policies, you can achieve the desired state of having no day-to-day need to use 'root'.

Some organizations use a 'security' or 'break-glass' IAM user with near-full administrative privileges to use in place of 'root' for most tasks. This can work well but you should still restrict access to the 'root' user to avoid any temptation to use it routinely.

For large organizations with complex AWS environments, implementing a 'Cloud Center of Excellence' or 'Cloud Custodian' operating model can help manage identity and access consistently across many AWS accounts.

## Explore further
- AWS IAM Best Practices - https://aws.amazon.com/iam/resources/best-practices/ 
- Video: Become an IAM Policy Master in 60 Minutes or Less - https://youtu.be/YQsK4MtsELU
- CIS Controls v8 - https://www.cisecurity.org/controls/ 
  - CIS Control 5.4 - Restrict Administrator Privileges to Dedicated Administrator Accounts