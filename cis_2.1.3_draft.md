# 2.1.3 Ensure all data in Amazon S3 has been discovered, classified and secured when required. (Manual)

## Summary
As an AWS user, it's crucial to make sure you have a handle on all the data stored in your S3 buckets. You need to know what data you have, how sensitive it is, and ensure it is adequately protected. Leveraging tools like Amazon Macie or third-party alternatives can help automate this process of data discovery, classification, and protection.

## Where did this come from?
This recommendation comes from the CIS Amazon Web Services Foundations Benchmark v3.0.0 - 01-31-2024. You can download a copy of the full benchmark from the [CIS website downloads page](https://downloads.cisecurity.org/#/).

The benchmark provides prescriptive guidance for configuring security options for a subset of Amazon Web Services with an emphasis on foundational, testable, and architecture agnostic settings. 

Additional information on protecting data in S3 can be found in the [S3 User Guide](https://docs.aws.amazon.com/AmazonS3/latest/userguide/security.html) and [Macie documentation](https://docs.aws.amazon.com/macie/latest/user/what-is-macie.html).

## Who should care? 
This is relevant for:
- Security engineers responsible for ensuring data in S3 is adequately protected
- Compliance officers that need to demonstrate data governance practices to auditors
- Developers storing potentially sensitive application data in S3 buckets

## What is the risk?
The main risk is sensitive data being stored in S3 without appropriate classification and protection applied. This could lead to:
- Data breaches if buckets are inadvertently made public or accessible to unauthorized parties
- Compliance violations if regulated data is not handled per requirements
- Reputational damage if customer data is leaked

Discovering and classifying data allows you to understand your risk exposure and apply protections like access control, encryption, and monitoring. Tools like Macie use machine learning to identify sensitive data types like personal information.

## What's the care factor?
If you are storing any sensitive, regulated, or customer data in S3, you should place a high priority on implementing this recommendation. A data breach can have severe consequences for your business.

Even if you don't believe you have sensitive data, it's worth conducting discovery to validate that assumption. You may be surprised what you find.

## When is it relevant?
This recommendation is most applicable when:
- Storing sensitive data like PII, PHI, financial records, IP in S3 
- Subject to compliance regimes like HIPAA, GDPR, PCI-DSS
- Lack visibility into the full scope of data stored in S3 across your organization

It's less critical if:
- Only using S3 for public, non-sensitive data like website assets
- Have a well-defined data classification scheme and SDLC process to protect sensitive data
- Already have an alternate DLP solution in place for S3

## What are the trade offs?
Discovering, classifying and securing data does come with some costs:
- Enabling Macie incurs additional charges based on the number of S3 buckets, amount and of data processed
- Some up-front effort is required to set up Macie jobs, review findings, and address issues
- Putting tight access controls on buckets may impact application functionality if not done carefully
- Classifying a large volume of existing data can be time-consuming 

However, these costs tend to pale in comparison to the impact of a major data breach.

## How to make it happen?
To enable Macie and start discovering data:

1. Log into the Macie console at https://console.aws.amazon.com/macie/
2. Click "Get started" then "Enable Macie"
3. Set up an S3 bucket to store sensitive data discovery results
  - Create a new S3 bucket or use an existing one  
  - Block all public access on the bucket
  - Configure encryption using KMS 
4. Create a job to discover sensitive data
  - In the Macie console, go to S3 buckets
  - Select which buckets you want to scan and click "Create job"
  - Give the job a name and description
  - Choose "Quick create" to run the job immediately or "Submit" to save for later
5. Review findings  
  - In the Macie console, go to Findings
  - Select a finding to view more details

For a more custom setup, refer to the [Macie User Guide](https://docs.aws.amazon.com/macie/latest/user/macie-user-guide.html).

## What are some gotchas?
- Macie requires IAM permissions to list and analyze data in the buckets you specify, so ensure the appropriate service role is configured
- Discovery jobs can take several hours to run depending on the amount of data to be scanned
- Macie cannot automatically analyze objects that are stored using the S3 Glacier storage classes
- By default, Macie uses the AWS managed CMK for S3 called "aws/s3" to encrypt sensitive data discovery results. If you want to use a customer-managed CMK instead, you must add certain permissions to the CMK policy.

## What are the alternatives?
Some other options for S3 data discovery and classification:

- Writing custom scripts to inventory S3 objects and look for sensitive data patterns (not recommended for most)
- Using open source tools like [Cloudcustodian](https://cloudcustodian.io/docs/aws/examples/index.html) or [Prowler](https://github.com/prowler-cloud/prowler)
- Deploying 3rd-party DLP solutions like Symantec, McAfee, Nightfall etc.

These may be preferable if you have a specific toolset you need to integrate with or want more customization. But for most AWS customers, Macie will be the simplest route.

## Explore further
- CIS Benchmark recommendation 2.1.2 for encrypting sensitive data at rest in S3
- [Macie FAQ](https://aws.amazon.com/macie/faq/) addresses common questions about the service
- [Using Macie to Detect Sensitive Data](https://aws.amazon.com/blogs/security/how-to-use-amazon-macie-to-preview-sensitive-data-in-s3-buckets/) blog post with more usage examples
- Consider integrating Macie findings with [Security Hub](https://aws.amazon.com/security-hub/?aws-security-hub-blogs.sort-by=item.additionalFields.createdDate&aws-security-hub-blogs.sort-order=desc) for a consolidated security view