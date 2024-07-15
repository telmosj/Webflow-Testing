# 1.12 Ensure credentials unused for 45 days or greater are disabled (Automated)

## Summary
It's important to keep a tight leash on your AWS IAM user credentials. Passwords and access keys that haven't been used in a while could spell trouble if they fall into the wrong hands. CIS recommends disabling or removing any credentials that have been gathering dust for 45 days or more. 

## Where did this come from?
This recommendation comes straight from the horse's mouth - the "CIS Amazon Web Services Foundations Benchmark v3.0.0 - 01-31-2024". You can download the full benchmark from the [CIS website](https://downloads.cisecurity.org/#/). The [AWS IAM best practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html#remove-credentials) docs also harp on about removing unnecessary credentials.

## Who should care?
This one is relevant for:
- IT administrators responsible for securing AWS accounts
- Security analysts monitoring AWS environments for risks
- Compliance officers ensuring adherence to security benchmarks 

## What is the risk?
The main risk here is compromised credentials. If an IAM user's password or access key is stolen or cracked, it could be used to access your AWS resources and wreak havoc. The longer those unused credentials hang around, the bigger the window of opportunity for baddies to exploit them.

Enforcing this 45 day disable/delete policy helps clamp down on that attack surface. It's especially important for credentials associated with abandoned or neglected IAM users that could fly under the radar.

## What's the care factor?
Admins should rank this one fairly high on the priority list. It's a pretty straightforward quick win from a security hardening perspective. 

Credential compromise can lead to data breaches, infrastructure tampering, and other expensive headaches. Snipping old credentials is a no-brainer to help prevent that.

## When is it relevant?
This recommendation should be applied to all IAM users across your AWS accounts. The only exception is the root account which is exempt from the 45 day purge.

There are very few situations where it wouldn't make sense - perhaps for break-glass emergency credentials that are rarely used but still important to have around just in case. But those should be tightly locked down with strong passwords and MFA.

## What are the trade offs?
The main effort involved is keeping tabs on credential age and usage across your IAM user base and proactively cleaning things up. Some manual oversight may be needed for edge cases.

Other than that, there shouldn't be major downsides. If credentials are truly unused, disabling/deleting them shouldn't crimp productivity. You may get some grumbles from users who stash access keys "just in case" but a bit of credential hygiene is worth it.

## How to make it happen?
Hunting for stale credentials is easy peasy through the AWS IAM console:

1. Login to the AWS Management Console 
2. Open the IAM service page
3. Click on "Users" in the navigation panel
4. Click the Settings cog icon
5. Select "Console last sign-in", "Access key last used", and "Access Key Id"  columns to view in user table
6. Check the "Console last sign-in" dates and look for any over 45 days old. "Never" means the user has never logged in.
7. Check "Access key last used" dates and look for any over 45 days old. "None" indicates a key has never been used.  
8. For any offending credentials, click on the the user, go to their Security Credentials tab and either deactivate the console password or delete the access keys.

You can also automate the stalking of stale credentials using the AWS CLI:

1. Generate an account credential report with:
   ```
   aws iam generate-credential-report
   ```
2. Download the credential report CSV file:
   ```
   aws iam get-credential-report --query 'Content' --output text | base64 -d > cred_report.csv
   ```
3. Check the `password_enabled`, `password_last_used`, `password_last_changed`, `access_key_1_active`, `access_key_2_active`, `access_key_1_last_used_date`, `access_key_2_last_used_date`, `access_key_1_last_rotated`, and `access_key_2_last_rotated` columns for each user
4. Deactivate/delete any credentials over the hill based on 45 day thresholds

## What are some gotchas?
The main permission needed to tinker with IAM user credentials is `iam:UpdateLoginProfile` to disable console passwords and `iam:DeleteAccessKey` to delete access keys. Your IAM admin role/user will need those.

Also, as mentioned above, the root account is exempt from this clean up crusade. But you shouldn't be using the root account regularly anyway! Follow best practices and lock that sucker down.

## What are the alternatives?
Instead of manually sifting through IAM credentials, you can use AWS Config rules like `iam-user-unused-credentials-check` to automatically alert you about credential geezers. That'll save some elbow grease.

You could also whip up a Lambda function to automatically deactivate/delete any 45 day old offenders. Pair it with a CloudWatch Events rule to run daily and keep your IAM nice and tidy.

## Explore Further
- AWS IAM Best Practices: https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html 
- AWS blog on finding unused credentials: https://aws.amazon.com/blogs/security/iam-access-analyzer-simplifies-inspection-of-unused-access-in-your-organization/
- CIS AWS Benchmark download: https://downloads.cisecurity.org/

This aligns with these other CIS Controls:
- 5.3 Disable Dormant Accounts (IG1, IG2, IG3)
- 16.9 Disable Dormant Accounts (IG1, IG2, IG3) - from the older CIS Controls v7