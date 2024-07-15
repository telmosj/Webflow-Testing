# 2.2.1 Ensure EBS Volume Encryption is Enabled in all Regions (Automated)

## Summary
Buckle up folks, it's time to talk encryption! AWS makes it easy to encrypt your Elastic Block Store (EBS) volumes, but for some reason it's not enabled by default. Flipping that switch is a quick win for improving your security posture and avoiding those dreaded "oopsie!" moments with your sensitive data.

## Where did this come from?  
This juicy tidbit of security wisdom comes straight from the "CIS Amazon Web Services Foundations Benchmark v3.0.0 - 01-31-2024", which you can download yourself at https://downloads.cisecurity.org/#/. The CIS Benchmarks are the gold standard for securely configuring all kinds of IT systems, and the AWS Foundations Benchmark zooms in on locking down your AWS environment. 

For the full scoop on EBS encryption, check out the AWS docs at https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSEncryption.html. And if you're the type who prefers your technical guidance in convenient blog form, feast your eyes on https://aws.amazon.com/blogs/aws/new-opt-in-to-default-encryption-for-new-ebs-volumes/. AWS really wants to make sure you don't miss out on the encryption party.

## Who should care?
- Cloud Engineers responsible for implementing secure AWS architectures** - You hold the keys to the kingdom (literally in this case!). Ensuring data-at-rest encryption is kind of a big deal.
- Compliance Officers who need to demonstrate security controls to auditors** - Unless you enjoy uncomfortable conversations with auditors, you'll probably want to make sure this encryption box is checked.  
- Security-conscious IT Leaders who don't enjoy seeing their company in headlines about data breaches** - Self explanatory really. Don't be that guy.

## What is the risk?
Here's the deal - if you leave your EBS volumes unencrypted, you're essentially leaving your data "naked" for anyone who gains unauthorized access to feast their eyes on. And let's face it, in today's cyber threat landscape, it's not a matter of if but when someone will try to get their grubby hands on your data. 

Encrypting those EBS volumes acts as a crucial last line of defense. Even if an attacker manages to access the data, without the encryption keys it will just look like unintelligible gibberish to them. Crisis averted!

Of course, encryption is not an absolute guarantee. If an attacker gets their hands on your encryption keys too, the jig is up. But it significantly reduces the blast radius of a data breach and buys you precious time to catch the issue and respond.

## What's the care factor?

Considering the potentially catastrophic financial and reputational damage of a major data breach, I'd say the care factor for encrypting your EBS volumes should be dialed up to 11 for most organizations. Unless you're just storing cat memes on those volumes (no judgement), this is not a security control you want to skimp on. 

Data breaches are expensive, not just from lawsuits and regulatory fines, but the harder-to-quantify costs of cleaning up your reputation and regaining customer trust. It's much cheaper to just flip the dang encryption switch.

## When is it relevant?

EBS encryption is a smart move anytime you are storing even remotely sensitive data on EBS volumes. Basically, if it's not something you'd want to post on a public GitHub repo, encrypt it! This is doubly true for organizations beholden to lovely compliance frameworks like PCI, HIPAA, GDPR, etc.

There may be a few edge cases where EBS encryption doesn't make sense, like:
- You're just storing public, non-sensitive data that you don't care if it leaks
- You are absolutely sure you'll never ever store anything sensitive on the EBS volume
- Compliance and privacy laws do not exist in your alternate universe

But in general, encryption is a pretty low-effort security win, so it's usually worth doing.

## What are the trade offs?

Of course, nothing in security is free. The main "costs" of enabling EBS encryption are:

- A bit more complexity to set up and manage, especially around key management
- A small performance hit (typically single digit percentage)  
- If you lose your encryption key you lose access to the data (so back those keys up!)

But all things considered, for most use cases these are pretty minor trade-offs for a substantial security boost. Performance is usually negligible with modern compute instances. And careful key management is just basic security hygiene.

## How to make it happen?

Thankfully AWS makes it pretty easy to make ubiquitous EBS encryption a reality. Here's how to make the magic happen:

Via Console:
1. Log into your AWS Management Console and pull up the EC2 Dashboard (https://console.aws.amazon.com/ec2/)  
2. Under "Account Attributes", click on  'Data Protection and Security'
3. On the "EBS encryption" section, click that big friendly "Manage" button
4. Tick the "Enable" checkbox to make encryption the default  
5. Click "Update EBS encryption" to seal the deal
6. Rinse and repeat for any other regions where you operate

Via CLI:
1. Open up that trusty terminal  
2. Run this for each region you need to lock down:
   `aws --region <region> ec2 enable-ebs-encryption-by-default`
3. Marvel at the `"EbsEncryptionByDefault": true` confirming your victory over unencrypted data

## What are some gotchas?

There are a couple key things to keep in mind:

- This only encrypts *new* EBS volumes by default going forward, existing unencrypted volumes stay unencrypted
- You'll need the right permissions to actually do this, specifically `ec2:EnableEbsEncryptionByDefault` and `ec2:DisableEbsEncryptionByDefault` (https://docs.aws.amazon.com/IAM/latest/UserGuide/list_amazonec2.html)
- If you use AMIs to launch new instances, make sure those AMIs are set to use encrypted EBS snapshots or you'll have to manually encrypt during launch

## What are the alternatives?

Really the main alternative to default EBS encryption is... manually enabling encryption on a per-volume basis. Which is better than nothing! But it's much easier to miss a volume that way. Hence why default encryption is the way.

I suppose you could also explore KMS plugins for more sophisticated key management with 3rd party providers, if that's your jam. But for most, AWS KMS does the trick.

## Explore further
If you're still hungry for more encryption knowledge, dig into these:

- AWS Key Management Service Deep Dive Whitepaper (https://docs.aws.amazon.com/pdfs/kms/latest/cryptographic-details/kms-crypto-details.pdf)  
- CIS Controls v8 3.11 - Encrypt Sensitive Data at Rest (for a broader take on encryption beyond just AWS)
- AWS re:Invent 2017: Encryption Everywhere: How to Achieve End-to-End Encryption (https://www.youtube.com/watch?v=gTZgxsCTfbk)

Go forth and encrypt!