# Ensure IAM password policy requires minimum length of 14 or greater

## Summary
It's important to enforce strong password policies in AWS to protect your accounts from unauthorized access. The Center for Internet Security (CIS) recommends setting a minimum password length of at least 14 characters for your AWS IAM password policy. This simple configuration change can significantly increase the difficulty for attackers to guess or brute-force passwords and gain access to your AWS resources.

## Where did this come from?
This recommendation comes from the CIS Amazon Web Services Foundations Benchmark v3.0.0 - 01-31-2024. You can download the full CIS benchmark from https://downloads.cisecurity.org/#/ for more details and related recommendations. The AWS documentation also provides guidance on setting IAM password policy at https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_passwords_account-policy.html.

## Who should care? 
This recommendation is relevant for:
- AWS account administrators responsible for configuring IAM policies and securing AWS accounts
- Security engineers tasked with hardening AWS environments and preventing unauthorized access
- Compliance officers ensuring adherence to security best practices and industry standards

## What is the risk?
Short, simple passwords are easy to guess or brute-force by attackers. If an attacker is able to obtain valid AWS IAM user credentials, they could gain unauthorized access to your AWS account and resources. This could lead to:
- Data breaches exposing sensitive information 
- Attackers spinning up expensive EC2 instances for crypto mining
- Deletion or tampering of critical resources disrupting your business
- Reputational damage and loss of customer trust

Setting a minimum 14 character password length significantly increases the difficulty and time required to crack passwords, reducing the likelihood of successful attacks.

## What's the care factor?
Administrators should treat this as a high priority recommendation to implement wherever possible. The increased security is well worth the minimal effort required to update the IAM password policy. However, balance security with usability - if you require excessively long or complex passwords, users may have difficulty remembering them or resort to insecure practices like writing passwords down.

## When is it relevant?
You should enforce a minimum 14 character password length for:
- All IAM users in production AWS accounts 
- IAM users with access to sensitive data and critical resources
- External IAM users like partners, vendors and contractors 

It may be acceptable to have a shorter minimum (but no less than 8) for:
- Test/dev accounts that don't contain production data
- Internal IAM users authenticating with single sign-on and MFA
- Application service accounts that use long random generated passwords 

## What are the trade offs?
Increasing password length requirements comes with some downsides:
- Users have to remember longer passwords which can be frustrating 
- Longer passwords take more time to type, impacting productivity
- When changing existing password policies, you may need to have users update passwords which creates work
- Some older applications may have hard limits on password length they can support

Overall, these are minor inconveniences compared to the security benefits in most cases. Provide guidance and password managers to help users cope.

## How to make it happen?
To set the IAM password policy to require a minimum of 14 characters, follow these steps in the AWS Management Console:

1. Sign in to the AWS Management Console and open the IAM console at https://console.aws.amazon.com/iam/
2. In the navigation pane on the left, click Account settings. 
3. In the Password policy section, click Edit.
4. Configure the password policy options as desired, ensuring Minimum password length is set to 14 or higher. 
5. Click Save changes.

You can also configure the password policy from the AWS CLI with:

```
aws iam update-account-password-policy --minimum-password-length 14
```

## What are some gotchas?
- The IAM password policy applies to all IAM users in the AWS account. You can't set different policies per user or group.
- Users with existing passwords less than 14 characters will be forced to change them to a compliant password on their next login after you set the policy.
- The policy only takes effect for the AWS account root user if they change their password. To force the root user to comply, set IAM User Access to Billing Information to Activate IAM Access then change the root password.
- To update the policy, you need permission for the iam:UpdateAccountPasswordPolicy action.

## What are the alternatives?
Some alternatives and complimentary practices to increase password security:
- Use AWS Single Sign-On with your corporate directory to enforce password policies consistently - https://docs.aws.amazon.com/singlesignon/latest/userguide/password-policies.html
- Require IAM users to use multi-factor authentication in addition to a password - https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_mfa_enable_virtual.html
- Use IAM Access Keys or temporary sessions generated with AWS STS instead of IAM user passwords where possible - https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html
- Implement a password expiration policy forcing periodic password resets - https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_passwords_account-policy.html

## Explore further
- CIS Amazon Web Services Foundations Benchmark v3.0.0 - 01-31-2024 - Recommendation 1.9 - Ensure IAM password policy prevents password reuse
- CIS Amazon Web Services Foundations Benchmark v3.0.0 - 01-31-2024 - Recommendation 1.10 - Ensure multi-factor authentication (MFA) is enabled for all IAM users that have a console password
- AWS IAM Best Practices - Lock Away Your AWS Account Root User Access Keys - https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html#lock-away-credentials
- CIS Controls v7 - Control 4 Controlled Use of Administrative Privileges - https://www.cisecurity.org/controls/controlled-use-of-administrative-privileges/