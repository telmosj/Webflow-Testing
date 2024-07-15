# 1.4 Ensure no 'root' user account access key exists (Automated)

## Summary
Hey there! Let's chat about a super important AWS security recommendation - making sure the almighty 'root' user account doesn't have any access keys. I know, it sounds a bit dull, but trust me, this one is crucial for keeping your AWS account safe and sound. So grab a snack and let's dive in!

## Where did this come from?
This sage advice comes straight from the pros at the Center for Internet Security (CIS). You can find it in the "CIS Amazon Web Services Foundations Benchmark v3.0.0 - 01-31-2024". If you want to read the whole enchilada, head over to https://downloads.cisecurity.org/# and search for the benchmark. For more juicy details, check out the AWS docs on [AWS Access Keys Best Practices](http://docs.aws.amazon.com/general/latest/gr/aws-access-keys-best-practices.html) and [Managing Access Keys](http://docs.aws.amazon.com/general/latest/gr/managing-aws-access-keys.html).

## Who should care?
This one is a biggie for:
- AWS account admins who want to keep things locked down tight 
- Security engineers on the hunt for potential holes to plug
- Compliance officers making sure everything is shipshape

## What is the risk?  
So here's the deal - that 'root' user account is like the master key to your entire AWS kingdom. If some sneaky hacker gets their hands on the access keys, they could wreak some serious havoc, like spinning up a bunch of expensive resources or stealing all your data. 

Deleting those 'root' access keys is like putting your master key in a maximum security vault - it drastically reduces the chances of your account getting compromised. Plus, it encourages you to set up proper user accounts with only the permissions they really need. 

## What's the care factor?
On a scale from "meh" to "drop everything and fix it now", this one rates a solid "seriously, do it ASAP". Those 'root' access keys are a golden ticket for baddies, so you don't want them floating around. The consequences of not following this recommendation could be catastrophic - imagine your company's data leaked or a massive unexpected bill. No bueno.

## When is it relevant? 
This recommendation is always relevant. No ifs, ands, or buts. Every AWS account should have the 'root' access keys deleted. Period.

The only time it wouldn't apply is if your account somehow doesn't have a 'root' user (which is pretty much impossible under normal circumstances).

## What are the trade-offs?
The good news is, deleting 'root' access keys is pretty low effort. It's a one-time task that takes just a few minutes. 

The only potential "downside" is that you'll no longer be able to use those keys to make programmatic calls to AWS. But that's the whole point! You shouldn't be using 'root' for day-to-day tasks anyway. Anything 'root' can do, a properly configured IAM user can do too.

## How to make it happen?
Alright, let's get down to brass tacks. Here's how to vanquish those 'root' access keys:

From the AWS Console:
1. Log in as the 'root' user and open the IAM console: https://console.aws.amazon.com/iam/  
2. Click on the <root_account> dropdown at the top right and select "My Security Credentials"
3. Click "Continue to Security Credentials" 
4. Go to the "Access Keys" section
5. If there are any active keys, click the "Delete" button (no take-backsies on this one!)

You can also use the CLI:
1. Run `aws iam get-account-summary`
2. Look for `"AccountAccessKeysPresent": 0` in the output
3. If you see a 1 instead of a 0, that means 'root' access keys exist
4. Use `aws iam delete-access-key` to delete the offending keys

## What are some gotchas?
The main thing to watch out for is accidentally deleting an access key that's still being used somewhere. Double check that you're deleting the right keys before you nuke 'em.

Also, keep in mind you'll need 'root' level permissions to delete the keys. If you're having trouble, make sure you're really logged in as 'root'.

For the CLI method, you'll need the `iam:GetAccountSummary` and `iam:DeleteAccessKey` permissions. 

## What are the alternatives?
Honestly, there's not really an "alternative" to deleting the 'root' keys. It's pretty much Security 101. 

I suppose you could rotate the keys regularly instead of deleting them, but that's just delaying the inevitable. Bite the bullet and delete 'em, I say!

## Explore further
If you want to dive deeper into the fascinating world of AWS access keys, check out:
- [IAM Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html) in the AWS docs 
- CIS Controls v8 3.3 and 5.4 for more info on data access control and restricting admin privileges

There you have it! Go forth and slay those 'root' access keys, my friend. Your AWS account will thank you.