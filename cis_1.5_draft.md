# 1.5 Ensure MFA is enabled for the 'root' user account (Automated)

## Summary
Hey there! Let's chat about a super important AWS security recommendation - enabling multi-factor authentication (MFA) for the root user account. This adds an extra layer of protection on top of the username and password. With MFA, after entering login credentials, you'll also need to provide a code from your MFA device. Better safe than sorry, right?

## Where did this come from?
This recommendation comes straight from the "CIS Amazon Web Services Foundations Benchmark v3.0.0 - 01-31-2024". You can download the full benchmark [here](https://downloads.cisecurity.org/#/). The benchmark provides a bunch of great AWS security best practices. For more details on managing the root user, check out the [AWS docs](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_root-user.html#id_root-user_manage_mfa).

## Who should care?
This one is critical for:
- AWS account owners responsible for the overall security of the AWS account 
- Security engineers tasked with implementing AWS security best practices
- Compliance officers ensuring the AWS environment meets regulatory requirements

## What is the risk?  
The root user has unrestricted access to your entire AWS account. If compromised, an attacker could wreak some serious havoc like stealing data, changing configurations, or racking up a giant bill. MFA makes it much harder for bad guys to gain access, even if they get their hands on the password. It's not foolproof, but a big step in the right direction.

## What's the care factor?
On a scale of 1-10, this one dials it up to 11. The root user is the keys to the kingdom. Unauthorized access could be catastrophic in terms of data loss, system downtime, reputational damage and more. Enabling MFA is a relatively low effort task for a high impact security win. It's a no-brainer!

## When is it relevant?
This should be implemented for every AWS account, no matter the use case. The only possible exception is for accounts that only contain test or dummy data with no sensitive info. But even then, it's good to build the MFA muscle memory. 

## What are the trade offs?
The main downside is a little extra friction in the login process. You'll need to have your MFA device handy whenever you want to login as root. This could be annoying in a pinch, but well worth the security benefits. Also, there's the risk of getting locked out if you lose the MFA device. Be sure to keep a backup!

## How to make it happen?
Alrighty, let's walk through it:

1. Login to the AWS Console as the root user
2. Go to the IAM dashboard 
3. Expand "Activate MFA on your root account" under "Security Status"
4. Click "Activate MFA"
5. Select "A virtual MFA device" and click "Next Step"
6. AWS will show you a QR code. Open up your virtual MFA app and either scan the QR code or manually enter the secret key
7. The MFA app will start generating codes. Type the first one in "Authentication Code 1". Wait 30 seconds for it to generate a new one and type that into "Authentication Code 2"  
8. Click "Assign MFA" and you're good to go!

For recommendations on virtual MFA apps, check out this [AWS doc](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_mfa_enable_virtual.html#enable-virt-mfa-for-root).

## What are some gotchas?
Not too many, this one is pretty straightforward. The main thing is you need to login as the root user to enable MFA, you can't do it with a regular IAM user. Also, be aware that the QR code is a representation of the secret key, which AWS doesn't store. So don't close the window until MFA is all setup!

One other note - it's best to use a dedicated MFA device that's kept charged and secure, rather than a personal device that might get lost or swapped out.

## What are the alternatives?
Instead of a virtual MFA app, you could use a hardware MFA device like a YubiKey. The high-level process is similar, you'd just choose "A hardware MFA device" in step 5. 

You can also require MFA for individual IAM users, which is a good supplement to this, but not a replacement for protecting the root user.

## Explore Further
- [AWS Doc on Enabling a Virtual MFA Device for Root](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_mfa_enable_virtual.html#enable-virt-mfa-for-root)
- [CIS V8 Control 6.5: Require MFA for Administrative Access](https://www.cisecurity.org/controls/v8/) (requires login)

This rec aligns with CIS V8 Control 6.5 which states "Require MFA for all administrative access accounts, where supported, on all enterprise assets, whether managed on-site or through a third-party provider." It's part of the CIS V8 Implementation Group 1.

Hope this helps explain why MFA on root is so critical and how to make it happen. Stay secure out there!