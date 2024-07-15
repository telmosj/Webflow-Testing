# 1.3 Ensure security questions are registered in the AWS account

## Summary
Hey there! Let's chat about an important step in securing your AWS account: setting up security questions. It's a simple but crucial task that can save you from a world of trouble down the line. By taking a few minutes to configure these questions, you'll be adding an extra layer of protection to your account.

## Where did this come from?
This recommendation comes straight from the "CIS Amazon Web Services Foundations Benchmark v3.0.0 - 01-31-2024". You can download the full benchmark from the [CIS website](https://downloads.cisecurity.org/#/). For more details on securing your AWS account, check out the [AWS Identity and Access Management (IAM) documentation](https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html).

## Who should care?
* AWS account owners who want to ensure the security of their root account
* Security teams responsible for maintaining the integrity of AWS accounts
* Developers and operations staff who need to access the root account in emergency situations

## What is the risk?
The root account in AWS has unrestricted access to all resources and services. If an attacker gains access to the root account, they could wreak havoc on your AWS environment, potentially leading to data loss, service disruptions, and financial damage. By setting up security questions, you add an additional authentication factor that can help prevent unauthorized access to the root account, even if the password is compromised.

## What's the care factor?
Securing the root account should be a top priority for anyone using AWS. While you should aim to use the root account as infrequently as possible, it's essential to have a secure failsafe in case you ever need to regain access. Investing a small amount of time in setting up security questions can save you from major headaches and potential catastrophes in the future.

## When is it relevant?
Setting up security questions is relevant in the following situations:
* When creating a new AWS account
* When performing a security review of an existing AWS account
* When the root account password or MFA token is lost or compromised

It may not be necessary to set up security questions if:
* You have already implemented a robust authentication process for the root account
* You have completely disabled the root account and rely solely on IAM users and roles

## What are the trade offs?
The main trade-off in setting up security questions is the time it takes to configure them. However, this is a one-time task that takes only a few minutes. Some users may find it inconvenient to remember the answers to the security questions, but this minor inconvenience is far outweighed by the added security benefits.

## How to make it happen?
1. Log in to the AWS Management Console using the root account credentials
2. Click on the root account name in the top right corner of the console
3. Select "My Account" from the dropdown menu
4. Scroll down to the "Configure Security Challenge Questions" section on the Personal Information page
5. Click on the "Edit" button
6. Select three security questions from the dropdown menus and provide unique, memorable answers for each
7. Click "Save questions" to apply the changes
8. Store the security questions and answers in a secure location, such as a password manager or a physical safe

## What are some gotchas?
To set up security questions, you must have root account access. If you have lost access to the root account, you may need to contact AWS support for assistance.

Ensure that the answers to your security questions are unique, memorable, and not easily guessable. Avoid using information that can be found on social media or through public records.

## What are the alternatives?
An alternative to setting up security questions is to implement a strong Multi-Factor Authentication (MFA) solution for the root account. This can include using a physical MFA device or a virtual MFA app. However, security questions can still provide an additional layer of security even when MFA is enabled.

## Explore further
* CIS Amazon Web Services Foundations Benchmark v3.0.0 - 01-31-2024
* AWS IAM Best Practices: https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html
* AWS Root Account Protection: https://docs.aws.amazon.com/accounts/latest/reference/root-user-tasks.html

This recommendation aligns with the following CIS Control:
* v8 17.2 - Establish and Maintain Contact Information for Reporting Security Incidents