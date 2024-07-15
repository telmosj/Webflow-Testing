# 1.6 Ensure hardware MFA is enabled for the 'root' user account (Manual)

## Summary
Hey there! Let's talk about a super important security recommendation from the smart folks at the Center for Internet Security (CIS). They say that for your AWS root account (the one with the most power), you should use a physical hardware device for multi-factor authentication (MFA) instead of just a virtual one. It's like adding a high-tech padlock to the front door of your AWS kingdom!

## Where did this come from?
This wisdom comes straight from the "CIS Amazon Web Services Foundations Benchmark v3.0.0 - 01-31-2024". You can download the full benchmark chock-full of other useful recommendations at [https://downloads.cisecurity.org/#/](https://downloads.cisecurity.org/#/).  The CIS benchmarks are considered the gold standard for securely configuring all kinds of IT systems, including AWS. You can learn more about MFA and securing AWS access in the [official AWS IAM documentation](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_mfa.html).

## Who should care?
This is a critical recommendation for:
- AWS account administrators with unrestricted access to resources
- Security engineers with responsibility for locking down AWS accounts 
- Compliance officers with requirements to implement strong MFA controls
- Executives with liability for costly breaches and security incidents

## What is the risk?  
Not enabling hardware MFA on your root AWS account leaves the keys to your kingdom vulnerable to unauthorized access through:
- Compromised passwords (e.g. brute force, phishing, reuse, sharing)
- Lost or stolen access keys and credentials   
- Malicious insider abuse of broad permissions

The root account has unrestricted permissions, so in the wrong hands it could allow attackers to silently modify, disrupt or destroy critical cloud infrastructure and data. hardware MFA provides an extra layer of defense even if credentials are compromised.

## What's the care factor?
For AWS accounts hosting any kind of important data or systems, administrators should care a LOT about this. The root account is so powerful that additional protection is essential. Regulators and security auditors will likely consider the lack of hardware MFA on root accounts to be a serious compliance gap. Applying it to at least high risk / high privilege accounts is a common security best practice.

## When is it relevant?
This recommendation is relevant for:
- All AWS accounts, but especially high risk production accounts 
- Accounts hosting sensitive, regulated or mission-critical data
- Enterprises with strong security, risk or compliance drivers

It may be OK to skip hardware MFA and use a virtual smartphone app for:
- Low risk test or demo accounts
- Accounts with no sensitive data or important infrastructure
- Individual learner or experimental accounts

## What are the trade offs?
The main downside to hardware MFA is the administrative overhead - someone needs to procure the physical devices, manage inventory as they are assigned to accounts, provision and deprovision them, and deal with lost tokens. At scale this can be a major undertaking. The devices also cost money (typically $20-80 each).

From a user perspective, carrying around a physical token is less convenient than an app. They can more easily be forgotten, lost, or temporarily misplaced leading to access issues. 

Overall though, for critical accounts, the strong security is usually worth the overhead and cost.

## How to make it happen?
Here's how to set up hardware MFA on the AWS root account:

1. Sign in to the AWS Console using your root account credentials. Note that you must use the actual root account - you can't configure its MFA using another identity.

2. Go to the IAM dashboard at https://console.aws.amazon.com/iam/. Click "Activate MFA on your root account" under "Security Status".

3. Click on "Activate MFA" 

4. Select "A hardware MFA device" and click "Next Step"

5. Enter the device serial number from the back of the physical token.

6. Enter the first code displayed on the device, wait 30 seconds, then enter the second code. You may need to press a button to get the device to cycle codes.

7. Click "Next Step" to complete the activation. 

The next time you log into the root account, you'll be prompted for a code from the hardware MFA device in addition to the password.

## What are some gotchas?
Some important things to keep in mind:
- You MUST use the root account itself to activate its MFA. Using other accounts, even with high privileges, will not work.
- The account must have IAM permissions to manage its own MFA device (e.g. `iam:EnableMFADevice`, `iam:DeactivateMFADevice`)  
- You'll typically need to procure and provision the hardware tokens which takes some admin effort. Common vendors are YubiKey, Gemalto, and RSA.
- Hardware tokens are easy to lose - have a process to revoke lost tokens ASAP.
- If you lose all MFA devices AND access keys for the root account, account recovery is difficult. Have a plan for this worst case.

## What are the alternatives?
If you can't use a physical hardware MFA device, a virtual MFA app on a smartphone is better than nothing. While not as secure as a dedicated device, it still provides an extra factor beyond a password. 

Some cloud-based identity providers can act as a "virtual MFA" mechanism using SMS, phone calls, or push notifications as second factors. However, these also have vulnerabilities compared to hardware tokens.

For lower-risk accounts, you may choose to accept these alternatives. But for anything important, hardware MFA on the root account is the way to go.

## Explore Further
- CIS Benchmark for AWS: https://www.cisecurity.org/benchmark/amazon_web_services
- Comparison of MFA options in AWS: https://aws.amazon.com/iam/features/mfa/
- How to enable MFA in AWS: https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_mfa_enable.html
- AWS Organizations and Service Control Policies: https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scp.html 

This CIS recommendation fits into the broader IAM CIS Control around using strong authentication mechanisms like MFA throughout an AWS environment. By protecting the powerful root account with highly secure hardware MFA, you can mitigate risks of catastrophic breaches and abuse.