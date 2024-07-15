# 3.1 Ensure CloudTrail is enabled in all regions (Automated)

## Summary
AWS CloudTrail is an essential service that records a detailed history of all API calls made in your AWS account. Enabling CloudTrail in all regions ensures you have complete visibility into activity across your entire AWS environment, even in regions you may not be actively using. Let's explore why this CIS recommendation is important and how to put it into practice.

## Where did this come from?
This recommendation comes from the CIS Amazon Web Services Foundations Benchmark v3.0.0 published on 01-31-2024. You can download the full CIS benchmark from the [Center for Internet Security website](https://downloads.cisecurity.org/#/). The benchmark provides prescriptive guidance for configuring security options across a wide range of AWS services.

For more background, check out the [AWS CloudTrail documentation](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-user-guide.html) and [this AWS Documentation](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/best-practices-security.html) on security best practices for Cloudtrail.

## Who should care?
Several key roles in an organization should pay close attention to this control:

- Security engineers responsible for monitoring and securing the AWS environment
- Compliance officers that need to demonstrate the organization is tracking all activity for audit purposes
- Incident responders who may need to investigate a security event and require a complete activity record

## What is the risk?
Without CloudTrail enabled consistently across regions, you face several risks:

- Attackers could perform malicious activity in unused regions that goes undetected 
- You lack the ability to audit all API activity for compliance reporting
- Incident investigations are hindered by gaps in the audit trail

While major incidents may be less likely in unused regions, applying this control globally helps manage those edge-case scenarios. It's an essential detective control.

## What's the care factor?
For most organizations, this should be a high priority, low-effort control to implement. Enabling CloudTrail in all regions is a one-time configuration that provides immediate security benefits. The costs are minimal compared to the compliance and incident response capabilities it unlocks.

It's a quick win that every AWS account should have in place from day one. Put it this way - you really don't want to be digging through a major incident and discover the critical audit trail you need simply doesn't exist because you hadn't turned on global CloudTrail. An ounce of prevention!

## When is it relevant?
This control makes sense for virtually every AWS account. Even if you only operate in a single region today, enabling it globally prepares you for future expansion and costs little.

About the only reason you might hold off is if you have a completely standalone account with no sensitive data that will never be used for anything real. But for any production environment, it's a must-have.

## What are the trade offs?
The main costs to consider are:

- Additional storage costs in S3 for the CloudTrail logs (usually pretty minimal)
- Increased cost for any log analysis you run across a larger volume of events
- Some added complexity to manage log storage lifecycle and protect logs appropriately

But for most organizations, these are far outweighed by the benefits of having a complete audit history.

## How to make it happen?

Enabling global CloudTrail via the console is fairly straightforward:

1. Open the [CloudTrail console](https://console.aws.amazon.com/cloudtrail)
2. Click "Trails" on the left navigation pane 
3. Click "Create Trail"
4. Enter a name for the new trail
5. Under "Trail settings" 
   - Select "Yes" next to "Apply trail to all regions" 
   - Choose an S3 bucket and folder prefix for storing the log files
6. Under "Event types" 
   - Select "Management events"
   - Select "All" under "API activity"
7. Review the settings and click "Create" to finish

You can also accomplish this via the AWS CLI:

```
aws cloudtrail create-trail --name my-trail --s3-bucket-name my-bucket --is-multi-region-trail --enable-log-file-validation
```

The key parameters are:

- `--is-multi-region-trail` to enable the trail across all regions
- `--enable-log-file-validation` to turn on log file integrity validation

To turn on global event logging for an existing trail:

```
aws cloudtrail update-trail --name my-trail --is-multi-region-trail
```

## What are some gotchas?
A few things to watch out for when implementing this control:

- Make sure you specify a valid S3 bucket that CloudTrail has permission to write to. The bucket policy needs to grant CloudTrail access. See the [AWS docs](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/create-s3-bucket-policy-for-cloudtrail.html) for an example.

- If you use a KMS key to encrypt log files, the CloudTrail service needs `kms:GenerateDataKey` and `kms:DescribeKey` permissions on the key. 

- Logs can accumulate significantly over time. Consider using S3 lifecycle policies to archive older logs to Glacier or expire them. 

- Remember to secure access to the S3 bucket containing the logs. Use IAM policies and bucket policies to restrict access to only authorized users/roles.

## What are the alternatives?
Frankly, there aren't many good alternatives to enabling CloudTrail on a global basis. It's really an essential core service. 

Some other options to consider in addition:

- Use AWS Config rules to monitor that global CloudTrail stays enabled 
- Stream CloudTrail events to an external security information and event management (SIEM) system for long-term retention and analysis

But CloudTrail should really be considered mandatory. It's the foundation that enables so many other security best practices.

## Explore further
Some additional resources to learn more about CloudTrail and related security practices:

- [Deep Dive on AWS CloudTrail](https://www.slideshare.net/slideshow/aws-cloudtrail-to-track-aws-resources-in-your-account-sec207-aws-reinvent-2013/28429633)
- [Security at Scale: Logging in AWS](https://d0.awsstatic.com/whitepapers/compliance/AWS_Security_at_Scale_Logging_in_AWS_Whitepaper.pdf)
- [CIS AWS Foundations Benchmark](https://www.cisecurity.org/benchmark/amazon_web_services/)
- CloudTrail can be used to support CIS Controls such as:
   - [6.2 Activate audit logging](https://www.cisecurity.org/controls/cis-controls-navigator/v7-1) 
   - [8.5 Collect Detailed Audit Logs](https://www.cisecurity.org/controls/cis-controls-navigator)

Hopefully this helps explain why the CIS Benchmark recommends enabling global CloudTrail, what the benefits are, and how to get it configured correctly in your AWS environment. It's a high-value security control every organization should have in place.