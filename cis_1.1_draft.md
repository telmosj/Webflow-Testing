# 1.1 Maintain current contact details (Manual)

## Summary
It's critical to keep your AWS account contact details up-to-date with email addresses and phone numbers for multiple people in your organization. This allows AWS to quickly reach out if they detect any suspicious activity or potential security issues with your account. Taking a few minutes to ensure your contact info is current can save major headaches down the road.

## Where did this come from?
This recommendation comes from the CIS Amazon Web Services Foundations Benchmark v3.0.0 - 01-31-2024. You can download the full benchmark document from the [CIS website](https://downloads.cisecurity.org/#/). The benchmark provides prescriptive guidance for configuring AWS accounts and services to align with security best practices. See the [AWS documentation on managing account settings](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/manage-account-payment.html#contact-info) for more details on updating contact info.

## Who should care? 
* AWS account owners who are responsible for the security of their AWS environment
* Security administrators who monitor AWS accounts for potential threats and respond to incidents
* Compliance managers who need to ensure AWS usage adheres to regulatory requirements and organizational policies

## What is the risk?
If your AWS account starts behaving in a suspicious or prohibited way, AWS will attempt to contact the account owner using the email addresses and phone numbers on file. If they can't reach anyone, and the behavior continues, AWS may proactively throttle traffic to/from your account. This could result in downtime and impaired service for your applications running on AWS. 

While rare, a compromised AWS account could be used to perform malicious or illegal activities like crypto mining, spam campaigns, or hosting offensive content. Prompt action is required to cut off bad actors.

## What's the care factor?
Keeping contact details current should be a top priority for all AWS account owners. It requires minimal effort but can make a huge difference in your ability to quickly respond if AWS observes problems with your account. Don't let stale contact info cause unnecessary downtime for your business!

The potential impact of not being reachable can be quite severe - up to and including AWS suspending your account entirely. An ounce of prevention is worth a pound of cure here.

## When is it relevant?
Maintaining current contacts applies to all AWS accounts across all industries and use cases. It's a fundamental responsibility of using AWS. 

Perhaps the only situation where it's less critical is for accounts that are tightly locked down and used in a very limited capacity, like a sandbox environment. But in general, you always want a way for AWS to reach you.

## What are the trade offs?
Honestly, there aren't many downsides to keeping contacts updated. It takes a small amount of work to set up email distribution lists and call forwarding, but once configured, maintenance is minimal. 

You do have to be comfortable sharing contact info with AWS. But they keep details confidential and will only use them to reach out about potential account issues - never for marketing or other purposes.

## How to make it happen?
1. Sign into the AWS Management Console
2. Open the Billing and Cost Management console 
3. Choose "Account" from the top menu
4. Click "Edit" next to Account Settings 
5. Update the email address and phone number under Contact Information. Use group aliases and call forwarding numbers if possible
6. Verify both the email and phone work by having AWS send a test notification
7. Click "Done" to save the changes
8. Validate contacts at least annually and any time your team has changes

## What are some gotchas?
The main requirement is that you need to be signed in as the root user or have IAM permissions to modify billing settings (aws-portal:*Billing). See the [AWS Billing documentation](https://docs.aws.amazon.com/IAM/latest/UserGuide/list_awsbilling.html) for the specific permissions involved.

When setting up email forwarding, be sure to include multiple team members so there's no single point of failure. But don't go overboard and spam a huge list every time AWS sends a notification.

For phone numbers, it's recommended to use a corporate number/call tree that routes to on-call staff or a network operations center. Avoid using any one individual's direct line or cell.

## What are the alternatives?
There's not really any alternative to keeping your contact info up-to-date in AWS. It's a required step if you want to maintain an AWS account in good standing. 

Some customers will set up a dedicated "security@" email alias and phone number for their AWS accounts. This makes it crystal clear how AWS should reach out and allows routing notifications to the right team.

## Explore further
* Read the [AWS account management documentation](https://docs.aws.amazon.com/accounts/latest/reference/manage-acct-creating.html) to learn more about updating alternate contacts, changing account info, and related tasks
* Understand the process for [what happens if your account is compromised](https://aws.amazon.com/premiumsupport/knowledge-center/potential-account-compromise/)
* Review the related CIS recommendation for enabling billing alarms to detect unusual spending (Recommendation 2.6)