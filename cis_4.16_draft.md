# 4.16 Ensure AWS Security Hub is enabled (Automated)

## Summary 
Hey there! Let's chat about AWS Security Hub. It's this nifty service that pulls together security info from all over your AWS accounts and services into one central place. Think of it like a big magnet, attracting security findings so you can easily see what's going on and focus on the important stuff first. Pretty slick, right?

## Where did this come from?
This recommendation comes straight from the "CIS Amazon Web Services Foundations Benchmark v3.0.0 - 01-31-2024". You can download the full benchmark over at the [CIS website](https://downloads.cisecurity.org/#/). The benchmark has a bunch of great suggestions for locking down your AWS environment. For more details on Security Hub specifically, check out the [AWS Security Hub User Guide](https://docs.aws.amazon.com/securityhub/latest/userguide/what-is-securityhub.html).

## Who should care? 
* Cloud Security Architects looking to centralize security monitoring
* DevOps Engineers wanting a unified view of misconfigurations and vulnerabilities 
* Compliance Officers needing to assess against security standards
* CISOs seeking situational awareness of overall AWS security posture

## What is the risk?
Without Security Hub enabled, you lose visibility into potential security issues lurking in your AWS environment. This means critical vulnerabilities, exposed resources, and deviations from best practice could go unnoticed. Attacks and breaches become more likely.

While Security Hub alone won't prevent incidents, it significantly helps manage risk by surfacing problems early. The sooner you spot an issue, the faster you can jump on it before bad things happen.

## What's the care factor?
On a scale of "meh" to "mega important", enabling Security Hub is definitely towards the top. It should be one of the first things you do in any new AWS account. 

The comprehensive, real-time insights it provides are invaluable for staying on top of security. Trying to manually collate findings from multiple services is a painful time-sink by comparison.

For orgs with data protection obligations or compliance requirements, Security Hub is a must-have. It helps demonstrate security due diligence and allows you to continuously evaluate against standards like CIS and PCI DSS.

## When is it relevant?
You should enable Security Hub in *every* AWS account and across *all* regions. It's relevant for practically everyone using AWS. The only exception would be extremely locked down, standalone accounts with no external connectivity. But those are pretty niche.

Size doesn't matter either. Whether you're running a simple personal project or a giant enterprise setup, Security Hub has value to add.

## What are the trade offs?
Thankfully, Security Hub is a lightweight service to enable. It doesn't really impact performance or the end-user experience. The main 'cost' is a bit of initial configuration effort and the ongoing work to triage/prioritize the findings it generates.

There are also some monetary costs as Security Hub is priced per AWS account per month. But unless you have a huge number of accounts, it's generally very affordable. Most folks would consider it a worthwhile investment given the benefits.

## How to make it happen?
Alright, let's get into the technical nitty-gritty of enabling Security Hub. 

### Console method:
1. Fire up your AWS Management Console and open the [AWS Security Hub](https://console.aws.amazon.com/securityhub/) console. 
2. In the top right corner, select a Region to configure Security Hub in.
3. If you see a "Security Hub > Summary" page, then good news - it's already enabled for this region! If you get a "Setup Security Hub" or "Get Started With Security Hub" page instead, proceed onwards.
4. Hit the glorious "Enable Security Hub" button.
5. Rinse and repeat steps 2-4 for any other regions you want to cover.

### CLI method:
If you're more of a command line hero, we can enable Security Hub with the AWS CLI too.

First, check the current Security Hub status in a region:
```
aws securityhub describe-hub
```

If it's already enabled, you'll see a 'SubscribedAt' date in the output. If not, you'll get an 'InvalidAccessException' error.

We can fix that by enabling Security Hub! To do it with the default standards:
```  
aws securityhub enable-security-hub --enable-default-standards
```

Or to enable it without the default standards:
```
aws securityhub enable-security-hub --no-enable-default-standards
```

Boom, done! Now you've got a lovely centralized view of security findings.

## What are some gotchas?
As with all things AWS, IAM is key. To get Security Hub up and running, you'll need an identity (user/role) with the right permissions. The most straightforward way is to attach the AWS managed policy `AWSSecurityHubFullAccess`. This policy includes the core `securityhub:*` permissions needed to enable and configure Security Hub.

Also, keep in mind that Security Hub requires AWS Config to be enabled in the account too. So make sure that's sorted beforehand or you'll hit a roadblock during setup.

## What are the alternatives?
Security Hub is great, but there are some other options out there:
* [Azure Security Center](https://azure.microsoft.com/en-au/services/security-center/) - Microsoft's equivalent for the Azure cloud platform
* [GCP Security Command Center](https://cloud.google.com/security-command-center) - Google's take on centralized security management
* [Splunk Enterprise Security](https://www.splunk.com/en_us/software/enterprise-security.html) - A popular on-prem SIEM for aggregating security data
* [Sumo Logic](https://www.sumologic.com/) - Another cloud-based SIEM alternative

## Explore further
* Read the [Intro to AWS Security Hub](https://docs.aws.amazon.com/securityhub/latest/userguide/what-is-securityhub.html) for an overview of core concepts and features.
* Dive deep with the [AWS Security Hub API Reference](https://docs.aws.amazon.com/securityhub/1.0/APIReference/Welcome.html) for info on automating Security Hub with the API/CLI.  
* For broader security guidance, check out the [CIS AWS Foundations Benchmark](https://www.cisecurity.org/benchmark/amazon_web_services/). It covers heaps of other great practices.
* Tune into the AWS Security Hub [Product Page](https://aws.amazon.com/security-hub/) and [FAQs](https://aws.amazon.com/security-hub/faqs/) to learn more.

That should give you a solid primer on AWS Security Hub and how to get it enabled. It really is a super useful service for wrangling security in the cloud. So go forth and centralize that security goodness!