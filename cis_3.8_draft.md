# 3.8 Ensure that Object-level logging for write events is enabled for S3 bucket (Automated)

Want to keep a closer eye on what's happening in your S3 buckets? Enabling object-level logging for write events will let you track all the juicy details of who's putting what where. It's like having your own personal S3 surveillance system!

## Where did this come from?
This recommendation comes straight from the super smart folks at the Center for Internet Security (CIS). Specifically, it's from the CIS Amazon Web Services Foundations Benchmark v3.0.0 released on 01-31-2024. You can download the full benchmark document from the [CIS website](https://downloads.cisecurity.org/#/).


## Who should care?
- Security analysts who need full visibility into S3 activity for monitoring and investigations 
- Compliance officers responsible for ensuring data handling meets regulatory requirements
- S3 administrators that want granular logging to track usage and troubleshoot issues

## What is the risk?
Without object-level logging enabled, you'll be flying blind if:
- An attacker gains access and exfiltrates sensitive data from S3
- A disgruntled employee intentionally deletes or corrupts critical files
- An application bug causes unintended modifications that lead to downtime

Enabling these extra logs ensures you have the audit trail needed to detect, investigate, and recover from S3 incidents. It's like an insurance policy for your data.

## What's the care factor? 
For most organizations using S3, I'd rate the importance of enabling object-level logging as high. The extra visibility is invaluable and the risk of missing something is too great.

The only exception would be if you have buckets that store completely non-sensitive public data. In that case, you may be able to justify skipping the extra logging to save on storage costs.

## When is it relevant?
Object-level logging makes sense anytime you're storing important data in S3 that requires monitoring. Some common use cases:
- Buckets containing sensitive customer PII 
- Data subject to compliance requirements like HIPAA, PCI, GDPR, etc.
- Logs and artifacts needed for security investigations
- Business-critical intellectual property and trade secrets

This feature is probably overkill for:
- Static website assets (e.g. images, css, js)
- Publicly shared datasets
- Temporary files that get purged regularly

## What are the trade offs?
No security control comes for free, and enabling object-level logging has some costs to consider:
- **Storage costs** - The extra logs take up space and increase your S3 bill. Make sure to lifecycle to cheaper tiers.
- **Performance hit** - Logging every object action adds a small amount of overhead. Probably not noticeable unless you have massive scale.
- **Log noise** - You may need to tune out uninteresting events so real security signals aren't lost in the noise.

## How to make it happen?
Ready to flip the switch on object-level logging? Here's how:

Using the AWS Console:
1. In the S3 dashboard, click the bucket you want to log 
2. Go to the "Properties" tab
3. Under "AWS CloudTrail data events" click "Configure in CloudTrail"
4. In the CloudTrail console, create a new trail or choose an existing one
5. Under "Data events" select "S3" from the dropdown
6. Choose "Log all events" (or pick specific buckets if you prefer)
7. Click "Update trail" to save the config
8. Rinse and repeat for other buckets as needed

From the CLI:
1. Use `put-event-selectors` to enable logging for a trail:

```
aws cloudtrail put-event-selectors --region <region-name> --trail-name <trail-name> --event-selectors '[{ "ReadWriteType": "WriteOnly", "IncludeManagementEvents":true, "DataResources": [{ "Type": "AWS::S3::Object", "Values": ["arn:aws:s3:::<bucket-name>/"] }] }]'
```

2. Modify the `--region` and bucket ARNs to match your setup
3. Specify multiple buckets or use a wildcard like `arn:aws:s3` to cover all buckets

## What are some gotchas?
A few things to watch out for when enabling object-level logging:
- Your CloudTrail trail must be set to "Enabled" and "Multi-region trail" to capture all events
- The IAM role for CloudTrail needs `s3:GetBucketAcl` and `s3:PutObject` permissions on the S3 bucket being logged
- Avoid logging to the same bucket you are auditing to prevent recursive madness
- Remember to enable encrypting logs with KMS for extra protection

## What are the alternatives?
If you just can't bring yourself to enable object-level logging, you have a few other options for monitoring S3:
- [Enable S3 server access logging](https://docs.aws.amazon.com/AmazonS3/latest/dev/ServerLogs.html) to track HTTP requests at the bucket level
- Use [S3 Event Notifications](https://docs.aws.amazon.com/AmazonS3/latest/dev/NotificationHowTo.html) to trigger alerts on specific object actions
- Set up [AWS Config](https://aws.amazon.com/config/) to track configuration changes to S3 buckets

## Explore further
Want to dive deeper into S3 security? Check out:
- [CIS Amazon Web Services Foundations Benchmark](https://www.cisecurity.org/benchmark/amazon_web_services/)
- [AWS Security Best Practices](https://aws.amazon.com/architecture/security-identity-compliance/)
- [S3 Security Best Practices](https://docs.aws.amazon.com/AmazonS3/latest/dev/security-best-practices.html)

Enabling object-level logging is just one piece of the S3 security puzzle. To really lock things down, make sure you're also:
- Encrypting data at rest and in transit
- Enforcing least privilege access with IAM policies 
- Enabling MFA Delete to prevent accidental bucket deletions
- Blocking public access unless absolutely needed

Stay vigilant and keep your S3 buckets safe!