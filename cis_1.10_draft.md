# 1.10 Ensure multi-factor authentication (MFA) is enabled for all IAM users that have a console password (Automated)

Multi-factor authentication (MFA) is like adding a deadbolt to your front door. It provides an extra layer of security beyond just a password, making it much harder for bad guys to break into your AWS account. Sure, it may take a few extra seconds to unlock that deadbolt, but isn't the peace of mind worth it? 

In this article, we'll dive into why MFA is so important, who should care about it, and how you can enable it on your AWS account. By the end, you'll see that setting up MFA is a no-brainer for keeping your cloud castle safe and sound. Let's get into it!

## Where did this come from?

This recommendation comes straight from the super smart folks at the Center for Internet Security (CIS). Specifically, it's part of the CIS Amazon Web Services Foundations Benchmark v3.0.0 released on 01-31-2024. You can download the full benchmark [here](https://downloads.cisecurity.org/#/).

The benchmark provides prescriptive guidance for configuring security options on your AWS account. It's basically a checklist of best practices. This particular recommendation, 1.10, is one of the most important ones in the bunch.

For more background, check out the [AWS Identity and Access Management User Guide](https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html) which has a whole section dedicated to MFA.

## Who should care? 

This is relevant for any AWS account owner or administrator. If you can sign into the AWS Management Console, you need MFA enabled on your account, period. 

More specifically, MFA should be mandatory for:

- IAM users with `AdministratorAccess` permissions 
- IAM users who can edit permissions and create/delete other IAM users
- Root account (although you should avoid using this anyway)

## What is the risk?

Without MFA, if a bad actor gets their hands on your AWS password, it's game over. They will have unfettered access to your entire AWS environment. 

This could lead to:

- Data breaches - attackers gaining access to sensitive data stored in S3, RDS, etc
- Crypto mining - attackers spinning up EC2 instances to mine cryptocurrency on your dime
- Ransomware - attackers encrypting your data and demanding payment
- Defacement - attackers modifying your website or application 
- Complete destruction - attackers terminating resources and deleting backups

The impact depends on what you have running in AWS, but it's safe to say it would be really, really bad across the board. An ounce of MFA prevention is worth a pound of incident response cure.

## What's the care factor?

Frankly, you should care a lot. Like "drop everything and set up MFA right now" level of caring.

Yes, there are a lot of security settings to configure. But if you had to prioritize just one, MFA would arguably be at the very top. It's such a simple and effective way to drastically reduce risk.

Still not convinced? Consider these factoids:

- The CIS Benchmark designates this as a Level 1 recommendation, meaning it's critical for security
- MFA is recommended as a best practice for interacting with sensitive and production AWS environments...by AWS themselves

This isn't just security fluff - it's the real deal. Ignore MFA at your own peril.

## When is it relevant?

MFA should be enabled for all IAM users that have a console password, all the time. No exceptions.

I know, never say never. But in this case, it's warranted. It doesn't matter if the account is for dev/test or production. Or if the user "only" has read permissions. If they have a password, they must have MFA. 

The only semi-exception is for machine users or bots/automation that use access keys. These don't have a console password, so they can't set up MFA. But really, you should considering using AWS IAM Roles instead of long-lived access keys anyway.

## What are the trade offs?

The biggest "cost" of MFA is the slight inconvenience of having to type in a code whenever you want to log into AWS. It adds maybe 10 seconds to the process. A small price to pay for a massive security upgrade.

The other potential challenge is dealing with lost/broken MFA devices. You'll need to have a process in place to revoke and re-issue credentials. But again, a small operational overhead compared to getting owned.

From a monetory perspective, if you use hardware MFA devices, there is a small upfront cost. But AWS supports virtual MFA which is 100% free using apps like Google Authenticator or Authy. Most organizations go the virtual route.

## How to make it happen?

Alright, let's get down to brass tacks. Here's how to enable MFA on an IAM user:

1. Sign into the AWS Console and navigate to IAM 
2. Select "Users" from the left sidebar
3. Click on the user you want to set up MFA for
4. Select the "Security credentials" tab
5. On the Multi-Factor authentication(MFA) tab, select 'Assign MFA device'
6. Provide a Device Name
7. Select "Authenticator App" (unless you have a physical one) and click "Continue"  
8. IAM will display a QR code. Open up your MFA app on your phone and scan the QR code to automatically configure it. If your app doesn't support QR, click "Show secret key" and manually type in the code.
9. The MFA app will start generating codes. Type the current code into "MFA code 1" in IAM. Wait 30sec for it to generate a new code and type that into "MFA code 2".
10. Click "Assign MFA" and you're done! 

Told you it was easy. Just repeat for any other IAM users. You also should [change the IAM password policy](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_passwords_account-policy.html) to require MFA for all users.

For the root account, the process is similar, except you need to log in as root first (which requires MFA if it's already enabled) and you'll use the "My Security Credentials" link in the top right instead of the IAM console.

## What are some gotchas?

Nothing too tricky, but a few things to keep in mind:

1. The user must have permission to manage their own MFA - the IAM policy needs `iam:EnableMFADevice`, `iam:ResyncMFADevice`, and `iam:ListMFADevices` permissions. The AWS managed policy `IAMUserChangePassword` includes this.

2. When you enable MFA for an access key (not console), it's tied to that specific access key. So if you rotate keys, you'll need to re-enable MFA.

3. Be very careful about recovery options. If a user gets locked out, you need a way to regain access. Either have multiple account admins, or consider using [IAM Access Analyzer](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-access-preview.html) to provide emergency access.

4. Some AWS services, like S3, have their own MFA delete protection. That's separate from IAM user MFA and needs to be configured in that service.

## What are the alternatives?

There really isn't a suitable alternative that gives you the same level of protection as true MFA. 

Some organizations try to get creative with things like jump hosts, split credentials across teams, or IP restrictions. Those are all fine and good, but more of a complement to MFA rather than a replacement. 

If you absolutely can't use MFA for some reason, at least make sure you:

- Use passwords that are long and complex (or better yet use a password manager)
- Enforce password expiration and prevent re-use 
- Implement a rock-solid offboarding process to revoke credentials immediately  
- Restrict logins to specific IP ranges

But really, just use MFA. It's easier. 

## Explore Further

Want to dive deeper? Here are some stellar resources:

- [AWS Multi-Factor Authentication](https://aws.amazon.com/iam/features/mfa/) - Overview page from AWS
- [Configuring MFA-Protected API Access](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_mfa_configure-api-require.html) - How to require MFA for API/CLI access to sensitive operations
- [Delegating MFA Management](https://aws.amazon.com/blogs/security/how-to-delegate-management-of-multi-factor-authentication-to-aws-iam-users/)