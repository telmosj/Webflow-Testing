# 1.20 Ensure that IAM Access analyzer is enabled for all regions (Automated)

## Summary
The AWS IAM Access Analyzer is a powerful tool that helps you identify resources in your AWS accounts that are shared with external entities. By enabling Access Analyzer in all regions, you can easily spot S3 buckets, IAM roles, and other resources that may be unintentionally exposed, allowing you to tighten up permissions and enforce least privilege access. It's like having X-ray vision for your AWS environment!

## Where did this come from?
This recommendation comes from the CIS Amazon Web Services Foundations Benchmark v3.0.0 released on 01-31-2024. You can download a copy of the full benchmark from the [CIS website](https://downloads.cisecurity.org/#/). 

The IAM Access Analyzer was first introduced at AWS re:Invent 2019. For more details on this service, check out the [Access Analyzer User Guide](https://docs.aws.amazon.com/IAM/latest/UserGuide/what-is-access-analyzer.html).

## Who should care? 
This matters most to:
- Security engineers responsible for ensuring least privilege access in AWS
- Cloud architects designing secure, well-governed AWS environments 
- Compliance officers validating adherence to security benchmarks and best practices
- Developers building apps that rely on AWS resources shared across accounts

## What is the risk?
Failing to enable IAM Access Analyzer or review its findings regularly could lead to:
- Data leakage from S3 buckets unintentionally shared publicly or with other accounts
- Privilege escalation through overly permissive IAM roles assumed by external principals 
- Compliance violations due to resources exposed outside of approved boundaries
- Costly data breaches resulting from compromised externally-facing resources

The IAM Access Analyzer won't prevent misconfigurations, but it will detect them and alert you quickly, minimizing exposure. Catching issues early is key to avoiding major incidents.

## What's the care factor?
For most organizations, this should be a high priority, especially if you:
- Store sensitive data in S3 or other services that support resource-based policies
- Federate access to AWS accounts or share resources with partners/vendors
- Are subject to compliance regimes like HIPAA, PCI-DSS, GDPR, etc.
- Have complex AWS environments with many accounts and regions

However, Access Analyzer may be less critical if you have a very simple setup, don't share resources externally, and tightly restrict access to AWS services. But it's still a good security hygeine practice.

## When is it relevant?
Turn on IAM Access Analyzer when you:
- Set up a new AWS account or Landing Zone 
- Undergo a compliance audit or security assessment
- Suspect a potential breach or notice unusual activity
- Grant access to AWS resources to a new partner, vendor or external party

It may be okay to leave it off in test/dev accounts with no production data. But for prod, it should always be on.

## What are the trade-offs?
Enabling IAM Access Analyzer is quite simple and low-effort. The main cost is remembering to review the findings regularly. Some potential downsides:
- In busy environments, alerts about exposed resources could become noise over time
- Admins may tune out analyzer findings if there are too many false positives 
- Fixing some resource exposures may break applications that depend on external access

But these are minor compared to the security benefits. You can adjust the alerting threshold to focus only on high-severity issues.

## How to make it happen?
To enable IAM Access Analyzer via console:

1. Open the IAM console and select "Access analyzer" 
2. Click "Create analyzer"
3. Confirm the target region (repeat this process for each region)
4. Name the analyzer (optional)
5. Add tags (optional) 
6. Click "Create"

To enable via CLI:

```
aws accessanalyzer create-analyzer --analyzer-name my-analyzer --type ACCOUNT
```

Repeat for each region, changing the --region parameter each time. 

Then set up a recurring calendar reminder to review Access Analyzer findings at least monthly. Create tickets to remediate any high-risk issues.

## What are some gotchas? 
To use IAM Access Analyzer, the account running it needs the following permissions:
- accessanalyzer:CreateAnalyzer
- accessanalyzer:CreateArchiveRule 
- accessanalyzer:GetAnalyzer
- accessanalyzer:GetArchiveRule
- accessanalyzer:ListAnalyzers
- accessanalyzer:ListArchiveRules
- accessanalyzer:ListFindings
- accessanalyzer:UpdateArchiveRule
- accessanalyzer:UpdateFindings
- iam:CreateServiceLinkedRole

Access Analyzer must also be able to reach the AWS endpoint in each region. Make sure you have connectivity and no interfering firewall rules.

Reference: [Access Analyzer IAM Policy Examples](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-policy-generation.html)

## Alternatives & Complements
Some other tools that can help manage resource policies:
- AWS Config - audit all resource configs against defined rules 
- Prowler - open source security assessment tool
- CloudSploit - identifies exposed resources across clouds
- Prisma Cloud - full CSPM solution across cloud providers

Use these along with IAM Access Analyzer for defense in depth.

## Explore Further
- Learn more about [resource-based policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_identity-vs-resource.html) and how IAM Access Analyzer secures them
- Understand the [access levels reported by IAM Access Analyzer](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-access-preview.html)
- Dive into the details of the [archive rules feature](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-archive-rules.html) for triaging results
- Examine the related CIS AWS Benchmark recommendations:
    - 1.16 - Ensure IAM policies are attached only to groups or roles
    - 1.22 - Ensure IAM policies that allow full "*:*" administrative privileges are not created
    - 2.9 - Ensure VPC flow logging is enabled in all VPCs