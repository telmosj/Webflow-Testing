# 1.2 Ensure security contact information is registered (Manual)

Hey there! Let's talk about an important but often overlooked security best practice - making sure your AWS account has up-to-date security contact information. I know, I know, paperwork is boring. But trust me, spending a few minutes on this now can save you major headaches later.

## Where did this come from?

This recommendation comes straight from the super smart folks at the Center for Internet Security (CIS). Specifically, it's from the [CIS Amazon Web Services Foundations Benchmark v3.0.0 - 01-31-2024](https://downloads.cisecurity.org/#/). Plenty more juicy security tips where this came from! For the AWS official word, check out the [AWS docs on managing alternate contacts](https://docs.aws.amazon.com/accounts/latest/reference/manage-acct-update-contact.html).

## Who should care? 

This one is key for:
- Cloud Security Managers responsible for incident response** 
- Compliance Officers ensuring adherence to industry standards**
- Paranoid IT Directors who stay up at night worrying about breaches**

## What is the risk?

Not having current security contacts on file with AWS means you could miss critical notifications in the event your account is compromised or if AWS detects suspicious activity. If bad things are going down, you want to be the first to know. Every second counts when stopping attackers in their tracks.

Think about it - if AWS notices something shady in your account, who are they gonna call? The number in their system. If that's the intern who quit 6 months ago... well you're gonna have a bad time. 

## What's the care factor?

On a scale from "meh" to "drop everything and do this now", this sits at a solid "schedule 5 minutes and knock it out this week". Not quite hair-on-fire urgent, but definitely do it ASAP. It's an easy win with high impact if something does go wrong.

## When is it relevant?

Keeping security contacts current is important for:
- Organizations of all sizes, from small startups to big enterprises 
- Teams using a significant amount of AWS services  
- Companies beholden to security compliance regimes like PCI, HIPAA, SOC2, ISO 27001, etc.
- Orgs who don't like surprises and prefer to be proactive about security

It may be less critical for:
- That random AWS account you use for testing 
- Completely isolated dev environments 
- Companies who enjoy living on the edge with a splash of danger

Basically, if you care at all about your production account security, this is for you.

## What are the trade offs?

Honestly, the trade-off here is pretty minimal - about 5 minutes of time from whoever manages the AWS root account. There's really no excuse not to do this.

I suppose you could argue there's a minor information disclosure risk by providing additional contact info to AWS. But c'mon now, if you can't trust AWS with a security@company.com email and a phone number, you got bigger problems my friend.

## How to make it happen?

Okay, rubber meet road. Here's how to actually implement this:

From the AWS Console:
1. Click your account name in the top right to open the dropdown
2. Click the "My Account" link
3. Scroll all the way down to the "Alternate Contacts" section 
4. Enter an email and phone number in the "Security" section
5. Click "Update" to save it
6. Treat yourself to a coffee for a job well done

Using the AWS CLI:
1. Make sure you have the CLI set up and permissions to modify account settings 
2. Run this command, replacing the placeholder values:
```
aws account put-alternate-contact --alternate-contact-type SECURITY --email-address security@company.com --name "John Doe" --phone-number "+1-555-123-4567"  --title "Security Manager"
```
3. Verify it saved by running:
```
aws account get-alternate-contact --alternate-contact-type SECURITY
```

Pro tip: Use a distribution list email like security@company.com so notifications get to the whole team. 

## What are some gotchas?

Not too many gotchas here, it's pretty straightforward. Just keep in mind:

- You need to be logged in as the AWS root user, or have IAM permission to modify alternate contacts. The required permission is `account:PutAlternateContact`.
- The security contact info is separate from the primary account contact and billing contact. Don't forget to keep those updated too.
- If you're using AWS Organizations, security contact info has to be configured in each member account. It doesn't inherent from the org master.

## What are the alternatives?

Um... not doing it? Living life on the edge? Hoping nothing ever goes wrong and you don't need AWS to notify you urgently? 

Yeah there's really no good alternative here. I guess you could setup your own fancy schmancy monitoring and alerting system for suspicious account activity. But why not just let AWS help with the heavy lifting for free?

## Explore further

Hungry for more? Here are some extra resources to dive deeper:

- User Guide: [Managing the AWS Accounts in Your Organization](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_accounts.html)
- [CIS AWS Foundations Benchmark](https://www.cisecurity.org/benchmark/amazon_web_services/) - The rest of the benchmark has 40+ more juicy security recommendations to sink your teeth into. Do 'em all and become an AWS security superhero.
- [AWS Security Incident Response Guide](https://docs.aws.amazon.com/whitepapers/latest/aws-security-incident-response-guide/welcome.html) - In case the worst happens, here's how to handle it like a pro.

That's it folks. Now stop procrastinating and go update those contacts. It'll take you less time than reading this article took. Your future self will thank you!