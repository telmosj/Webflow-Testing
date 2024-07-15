# 1.9 Ensure IAM password policy prevents password reuse

## Summary
Reusing passwords across multiple sites or systems is a bad habit that can seriously jeopardize your security. That's why AWS allows you to configure your IAM password policy to prevent users from cycling through old passwords. By requiring a history of 24 unique passwords before allowing a password to be reused, you can help protect your AWS environment from password spraying and brute force attacks.

## Where did this come from?
This recommendation comes from the CIS Amazon Web Services Foundations Benchmark v3.0.0 published on 01-31-2024. You can download a copy of the full benchmark from the [CIS website](https://downloads.cisecurity.org/#/). The benchmark provides prescriptive guidance for configuring AWS security options. For more details on password policies, check out the [AWS IAM documentation](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_passwords_account-policy.html).

## Who should care? 
* Security engineers responsible for defining authentication policies and protecting AWS accounts from unauthorized access
* Compliance officers who need to ensure IAM configuration aligns with industry best practices and standards
* IT managers overseeing a migration to AWS or standing up new environments in the cloud

## What is the risk?
The main risk of allowing password reuse is that it enables attackers to more easily gain unauthorized access to accounts. Some key risks:

* Password spraying attacks - Attackers use lists of common passwords across many different accounts. Reusing a breached password makes this much easier.
* Brute force attacks - Attackers systematically try all possible passwords for an account. Forcing long, complex, unique passwords makes cracking exponentially harder.
* Compliance failures - Many security standards require preventing password reuse as a basic control. Allowing reuse could mean failing your next audit.

Properly configured password policies, including password reuse prevention, can effectively mitigate these risks. While not a silver bullet, this control is an important part of a defense-in-depth strategy.

## What's the care factor?
Password policies are security hygiene 101. They are simple to implement, have relatively low user impact, and address major risks. Every organization should have them in place. 

However, the priority you place on this particular setting depends on a few factors:
* User maturity - If you have highly technical users following other security best practices, this is less critical than if you have novice users likely to pick weak passwords if allowed.
* Sensitivity - The more sensitive the data and systems being protected, the more important it is to implement strict password policies.
* Existing controls - If you already have strong MFA, this becomes less important. But it's still a good idea.

Overall this is an easy win all around and there's not a great reason not to do it. Make it a priority to close this gap if you haven't already.

## When is it relevant?
Password policies should apply to all IAM users. However, a few use cases to consider:

* Service accounts - Prevent reuse is less important here since they are non-interactive. But it's still a good practice.
* Break glass scenarios - You may need to override password history requirements to allow an admin to regain access in an emergency. But this should only happen in exceptional situations.
* Newly created accounts - When you create a new AWS account, configure this setting before provisioning any IAM users. There's no reason not to start off on a strong footing.

## What are the trade offs?
Preventing password reuse means your users have to be more creative. Some potential downsides:

* User experience - Some users may be annoyed at having to come up with new passwords often. This is especially true if you also require very long, complex passwords. Consider pairing this with a password manager.
* Support overhead - Prepare for an uptick in tickets for password resets, especially at first. Make self-service password reset available if you can.
* Loss of access - Rarely, a user may forget their password, get locked out, and not be able to recall any of their last 24 passwords. Have a plan in place for secure identity verification to regain access if needed.

The enhanced security is well worth these fairly minor trade-offs in most cases. But it's important to consider the impact, communicate the changes, and give your support teams a heads up before rolling it out.

## How to make it happen?
You can update your IAM password policy through the AWS Console or CLI.

To prevent password reuse in the AWS Console:

1. Sign in to the AWS Management Console and open the IAM console at https://console.aws.amazon.com/iam/
2. In the navigation pane, click Account settings. 
3. In the Password policy section, select Prevent password reuse.
4. For Number of passwords to remember, enter 24.
5. Click Apply password policy.

To prevent password reuse from the AWS CLI, run this command:

```
aws iam update-account-password-policy --password-reuse-prevention 24
```

If you have other password policy requirements, like minimum length, you can include those in the same command:

```
aws iam update-account-password-policy --minimum-password-length 14 --require-symbols --require-numbers --require-uppercase-characters --require-lowercase-characters --password-reuse-prevention 24
```

Refer to the [update-account-password-policy documentation](https://docs.aws.amazon.com/cli/latest/reference/iam/update-account-password-policy.html) for details on all the available parameters.

## What are some gotchas?
A few things to watch out for when configuring password reuse prevention:

* Root user - The setting governs the root user as well. Consider using a password manager for the root account since that password may not be changed often.
* Existing users - The policy only takes effect for new passwords set after applying it. It won't check a user's password history retroactively. Consider requiring a password reset to ensure uniqueness.
* Permissions - You need the `iam:UpdateAccountPasswordPolicy` permission to set this option. Make sure the IAM user or role you use to configure it has the proper rights.
* Other settings - The AWS console has a handy "Apply password policy" button that configures all the recommended password settings. But if you use the CLI, pay attention that you're setting `--password-reuse-prevention` and not `--allow-users-to-change-password` or another option by mistake.

## What are the alternatives?
Preventing password reuse is just one piece of the authentication puzzle. A few other key things to consider:

* Multi-factor authentication - Enforce MFA on user accounts, especially admins. Ideally use a hardware-based solution like a YubiKey.
* Password expiration - AWS doesn't allow forcing periodic password changes, but you can use third-party identity providers to achieve this if needed for compliance.
* Password complexity - Raise the required complexity to a high bar. At least 14 characters including uppercase, lowercase, numbers, and symbols.
* Password age - AWS keeps a password history of the last 24 passwords indefinitely by default. But you can adjust this duration using the `--password-reuse-prevention-history-days` CLI flag.

Combine reuse prevention with these other tools to improve your AWS authentication posture. While nothing is foolproof, these controls significantly raise the bar for attackers.

## Explore further
* CIS AWS Foundations Benchmark Recommendation 1.8 - Ensure IAM password policy requires at least one lowercase letter
* CIS AWS Foundations Benchmark Recommendation 1.10 - Ensure multi-factor authentication (MFA) is enabled for all IAM users that have a console password
* [AWS IAM Best Practices Guide](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html) 
* [OWASP Password Storage Cheat Sheet](https://github.com/OWASP/CheatSheetSeries/blob/master/cheatsheets/Password_Storage_Cheat_Sheet.md)
* [NIST Digital Identity Guidelines](https://pages.nist.gov/800-63-3/)