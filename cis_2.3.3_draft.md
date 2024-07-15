# Ensure that public access is not given to RDS Instance (Automated)

## Summary
Hey there! Let's chat about a super important security recommendation for your AWS environment - making sure your RDS database instances aren't open to just anyone on the internet. It's a simple config change that can save you from some major headaches down the road. Ready to dive in?

## Where did this come from?
This sage advice comes straight from the pros at the Center for Internet Security (CIS). It's part of their "CIS Amazon Web Services Foundations Benchmark v3.0.0" released on 01-31-2024. You can grab the full benchmark over at https://downloads.cisecurity.org/#/ to get all the nitty gritty details. The kind folks at AWS also have some great supplemental docs on locking down your RDS instances here: https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/UsingWithRDS.html

## Who should care? 
This one is crucial for any Database Administrator or DevOps Engineer responsible for provisioning and securing RDS database instances in AWS. If that's you, listen up!

## What is the risk?
Leaving your RDS databases open to public access is like leaving your front door wide open with a big neon "FREE STUFF" sign. It's an open invite for hackers to waltz right in and cause chaos. We're talking potential brute force attacks, nasty SQL injections, or even DDoS attacks that could take you offline. No bueno.

## What's the care factor?
On a scale from "meh" to "mega important", this one rates pretty darn high for the folks in charge of RDS security. Those risks we just talked about? They're not just theoretical. Breaches happen every day because of misconfigurations like this. A few minutes spent tightening up access could literally save your bacon (or at least your job).

## When is it relevant?
This CIS recommendation should be applied to each and every RDS instance you provision, no matter how big or small. The only potential exception *might* be for a database used by a public-facing app that absolutely needs to be accessible from anywhere. But even then, you'd want to be dang sure your security settings are bullet-proof. When in doubt, lock it down.

## What are the trade offs?
The main "cost" of implementing this is really just the time it takes your database admin to make the configuration change. There shouldn't be any real impact on performance or user experience, since you're likely already using security groups and VPCs to control access. The upside of better security far outweighs any minor inconvenience.

## How to make it happen? 
Alrighty, let's get down to brass tacks. Here's how you can check and change this setting:

1. Log into your AWS Management Console and open the Amazon RDS dashboard. 
2. Find the list of your database instances and click on the one you want to check.
3. Look under the "Connectivity & security" tab and check the "Public accessibility" setting. If it says "Yes", keep following these steps.
4. Click the "Modify" button up top to edit the instance settings.
5. Scroll down to "Network & Security" and expand the "Additional connectivity configuration" section. 
6. Set "Public accessibility" to "No".
7. While you're here, double check the security group assigned to control access to the VPC. Make sure it's not open to the world.
8. Schedule the change to happen immediately or during your next maintenance window and click "Modify DB Instance" to save it.

Boom, you're done! Repeat those steps for any other RDS instances in your environment.

## What are some gotchas?
The main thing to be aware of is that you'll need to have the right IAM permissions to make this change. Your database admin will need something like  `rds:ModifyDBInstance` to edit RDS settings. Check out the full list of actions for RDS here: https://docs.aws.amazon.com/service-authorization/latest/reference/list_amazonrds.html

Also, in some cases, the "Publicly accessible" flag might already be set to "No" but your RDS instance could still be in a public subnet with an internet gateway. So don't just rely on that one setting - make sure to check the VPC/subnet configs too.

## What are the alternatives?
Really the only alternative to restricting public access is... not restricting public access. Which we already decided is a bad idea, remember? I suppose you could set up a EC2 bastion host as a gateway for database admins to access RDS instead of opening it directly. But you're still relying on the security group configs to control the access. Probably best to just keep it simple and lock things down directly in RDS.

## Explore Further
If you really want to dive deep on RDS security, check out these docs from AWS:

- Working with a DB instance in a VPC: https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_VPC.WorkingWithRDSInstanceinaVPC.html 
- More RDS FAQs: https://aws.amazon.com/rds/faqs/

Tying it all back to the CIS benchmark, locking down public RDS access maps to a couple of key controls:

- [CIS Control 3.3 "Configure Data Access Control Lists"](https://www.cisecurity.org/controls/cis-controls-navigator) - Gotta make sure those databases access permissions are ship-shape!

And there you have it - a whirlwind tour through CIS Recommendation 2.3.3. Hopefully this helps you keep your RDS instances (and your job) safe and sound. Now go forth and secure all the things!