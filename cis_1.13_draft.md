# 1.13 Ensure there is only one active access key available for any single IAM user (Automated)

## Summary
Hey there! Let's chat about access keys in AWS. These little guys are super important for accessing AWS resources programmatically. But here's the thing - you really should only have one active access key per IAM user. It's just good security practice, you know?

## Where did this come from?
This stellar advice comes straight from the "CIS Amazon Web Services Foundations Benchmark v3.0.0 - 01-31-2024". You can grab a copy of the benchmark over at https://downloads.cisecurity.org/#/. The CIS benchmarks are chock-full of great security recommendations for AWS. If you want to dive deeper into access keys, check out the [AWS docs on access key best practices](https://docs.aws.amazon.com/general/latest/gr/aws-access-keys-best-practices.html) and [managing access keys](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html).

## Who should care? 
This is a biggie for:
- Security engineers with a passion for locking down AWS accounts
- DevOps pros who manage IAM users and want to keep things ship-shape
- Compliance officers tasked with making sure AWS accounts meet security standards

## What is the risk?
Multiple active access keys floating around for a single user is just asking for trouble. It increases the risk of accidental exposure and makes it tougher to track down issues if a key is compromised. Imagine the headache of trying to figure out which key was leaked when there are a bunch in play. No thanks!

Sticking to one active key per user helps prevent unauthorized access and makes incident response a whole lot smoother if a key does get into the wrong hands.

## What's the care factor?
If you fit into one of those target audiences up there, you should definitely care about this one. It's a low-effort, high-impact way to boost your AWS security posture.

For security and compliance folks, it's a must-have to meet baseline security standards. For DevOps teams, it's just good housekeeping to keep access keys tidy and under control.

## When is it relevant?
This recommendation is relevant for pretty much all AWS accounts that use IAM users and access keys. The only time it might not apply is if you have an IAM user that doesn't need programmatic access at all.

## What are the trade offs?
Honestly, there aren't a ton of downsides to this one. It might be a slight inconvenience to track down and deactivate extra keys, but it's well worth the security benefits.

The main "cost" is the time and effort to audit your IAM users and clean up any unnecessary active keys. But once you've done that initial sweep, it's easy to maintain going forward.

## How to make it happen?
Alrighty, let's get into the nitty-gritty of how to actually implement this recommendation.

From the AWS Console:
1. Sign in to the AWS Management Console and head over to the IAM dashboard (https://console.aws.amazon.com/iam/)
2. In the left navigation panel, choose "Users"
3. Click on the IAM user you want to check on
4. On the user details page, go to the "Security Credentials" tab
5. Under the "Access Keys" section, take a gander at the "Status" column 
6. If the user has more than one access key with a status of "Active", it's time to do some cleanup
7. Choose the access key you want to keep (ideally one that's less than 90 days old) and make sure it's working with your applications
8. For the other active access key(s), click "Make Inactive" to disable them
9. Rinse and repeat for each IAM user

From the CLI:
1. Use the `aws iam list-users` command to get a list of all your IAM users:
```
aws iam list-users --query "Users[*].UserName"
```
2. For each user, run `aws iam list-access-keys` to see their active access keys:
```
aws iam list-access-keys --user-name <user-name>
```
3. If a user has multiple active keys, choose the one to keep (less than 90 days old, tested with your apps)  
4. Deactivate the other active key(s) with:
```
aws iam update-access-key --access-key-id <access-key-id> --status Inactive --user-name <user-name>  
```
5. Confirm the keys are inactive with `aws iam list-access-keys --user-name <user-name>`
6. Keep on trucking through each IAM user

## What are some gotchas?
A couple things to keep in mind:
- You'll need `iam:ListAccessKeys` and `iam:UpdateAccessKey` permissions to pull this off
- Before you deactivate a key, make sure it's not being used by any of your applications (that's where having a key rotation strategy comes in handy)
- If you accidentally deactivate the wrong key, no worries - you can always reactivate it

## What are the alternatives?
In terms of achieving the same goal of limiting active access keys, there aren't a ton of alternatives. You could go scorched earth and delete extra access keys instead of just deactivating them, but that's a bit extreme.

If you want to get fancy, you could look into using [AWS Access Analyzer](https://docs.aws.amazon.com/IAM/latest/UserGuide/what-is-access-analyzer.html) to help identify unused or unnecessary access keys. But at the end of the day, you'll still need to manually clean them up.

## Explore further
If you want to level up your access key game, check out these resources:
- [AWS Access Keys Best Practices](https://docs.aws.amazon.com/general/latest/gr/aws-access-keys-best-practices.html) 
- [Rotating Access Keys](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html#Using_RotateAccessKey)
- [CIS AWS Foundations Benchmark](https://www.cisecurity.org/benchmark/amazon_web_services) for more juicy security recommendations

This recommendation aligns with:
- CIS Controls v8 5 - Account Management 
- CIS Controls v7 4 - Controlled Use of Administrative Privileges

And that's the scoop on keeping your access keys in check! Remember, when it comes to active access keys - less is more. Keep it to one per user and you'll be sitting pretty from a security perspective. Happy key wrangling!