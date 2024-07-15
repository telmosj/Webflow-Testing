# 3.9 Ensure that Object-level logging for read events is enabled for S3 bucket (Automated)

Want to keep a close eye on what's happening in your S3 buckets? Enabling object-level logging for read events is a must. This handy feature logs API operations like GetObject, so you can see exactly who's accessing your data and when. It's a key part of meeting compliance requirements and performing thorough security analysis.

## Where did this come from?

This recommendation comes straight from the "CIS Amazon Web Services Foundations Benchmark v3.0.0 - 01-31-2024". You can download the full benchmark over at https://downloads.cisecurity.org/#/ for all the juicy details. The kind folks at the Center for Internet Security (CIS) put a lot of thought into these guidelines to help keep your AWS environment secure.

For some extra background, check out the [AWS CloudTrail docs](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/logging-data-events-with-cloudtrail.html) on logging data events. They dive into how CloudTrail integrates with S3 to enable object-level logging.

## Who should care? 

This one's for all you:
- Security analysts with a passion for tracking down suspicious behavior 
- Compliance officers with a keen eye for meeting data regulations
- Curious S3 admins who want the full scoop on bucket activity
- Enthusiastic auditors giddy about generating detailed access reports

## What is the risk?

Without object-level logging enabled, you're flying blind when it comes to read activity in your S3 buckets. If an unauthorized user gains access and starts snooping around your sensitive data, you might never know. This could lead to:

- Data breaches that damage your company's reputation 
- Compliance violations and hefty fines
- Inability to properly investigate security incidents
- Undetected data exfiltration by malicious insiders

Enabling this feature significantly improves your detective capabilities. You'll have extensive logs showing every GetObject request. This makes identifying unauthorized access attempts and tracing the source much easier.

## What's the care factor?

I'd rate the care factor for this pretty high, especially if your S3 buckets contain sensitive data. Object-level logging is a foundational detective security control. 

For buckets with publicly accessible data, it may be less critical since you expect a high volume of read events from untrusted sources. Even then, logging can help identify any unusual usage patterns that may be concerning.

## When is it relevant?

Object-level logging for read events makes a lot of sense when:

- You have private S3 buckets storing sensitive data 
- Industry or government regulations require logging all data access
- You want to perform detailed monitoring and analytics on S3 usage
- Your S3 usage follows the standard request-response model

It may be less applicable if:

- Your S3 bucket only contains public data anyone can read
- You're using S3 as a static website hosting or content distribution platform
- Logging every read event would generate an unmanageably high volume of data
- The additional cost of storing and analyzing CloudTrail logs is prohibitive

## What are the trade offs?

The main downside of enabling object-level logging for reads is the potential cost. Depending on your usage patterns, it can generate a huge number of events. You'll need to pay for CloudTrail to log all those events, and any log analysis or SIEM tools to process them. 

There are also some nuanced performance implications. Technically, CloudTrail has to process each S3 request to determine if it needs logging which adds a small overhead. For most use cases this is negligible.

Finally, someone will need to actually monitor the logs and investigate any issues. That's additional time and effort for your security team. Though I'd argue that's time well spent.

## How to make it happen?

We can tackle this through the console or CLI, whichever tickles your fancy.

### Through the Console

1. Sign into the AWS Management Console and open the CloudTrail dashboard
2. Select Trails from the navigation panel on the left
3. Choose the trail you want to configure from the list 
4. On the trail details page, scroll down to the "Data events" section
5. Under "Data event type" select "S3" from the drop-down
6. For "Log selector template" choose "All current and future S3 buckets"  
7. Check the box next to "Read" under "Event types"
8. Click "Save" and you're all set!

You'll need to repeat this process for each region you want to enable object-level logging in.

### From the Command Line

1. Use the `describe-trails` command to list available CloudTrail trails:

```
aws cloudtrail describe-trails --region <region-name> --output table --query trailList[*].Name
```

2. Pick the trail name you want to update and plug it into this `put-event-selectors` command:

```
aws cloudtrail put-event-selectors --region <region-name> --trail-name <trail-name> --event-selectors '[{ "ReadWriteType": "ReadOnly", "IncludeManagementEvents":true, "DataResources": [{ "Type": "AWS::S3::Object", "Values": ["arn:aws:s3"] }] }]'
```

That `--event-selectors` bit is where the magic happens. We're telling CloudTrail to log read events for all current and future S3 buckets.

Rinse and repeat for each region as needed.

## What are some gotchas?

The big one is permissions. To enable object-level logging, the IAM user or role making the configuration change needs the right permissions. Specifically, these actions:

- `cloudtrail:DescribeTrails`
- `cloudtrail:GetEventSelectors`
- `cloudtrail:PutEventSelectors`

The CloudTrail documentation has the full scoop on [required permissions](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/security_iam_id-based-policy-examples.html).

You'll also want to consider your overall logging strategy. Cranking up the verbosity without proper log management can quickly get messy and expensive. Make sure you have a plan for storing, analyzing, and archiving all those juicy CloudTrail logs.

## What are the alternatives?

Sadly, there's no simple alternative that provides the same granularity as CloudTrail's object-level logging. 

You could enable S3 server access logging, which tracks requests made to a bucket. But that's all requests, not just read events, and the log format is different.

There are also third-party security tools that can monitor S3 activity. They usually rely on CloudTrail as the data source though, so you're back to square one.

At the end of the day, CloudTrail is the way to go for detailed object-level logging in S3. Embrace it!

## Explore further

Still curious? Here are some helpful resources to continue your S3 logging adventures:

- [AWS CloudTrail User Guide](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-user-guide.html)
- [Monitoring Amazon S3 Using CloudTrail Events](https://docs.aws.amazon.com/AmazonS3/latest/userguide/cloudtrail-logging-s3-info.html)


This recommendation aligns with [CIS Control 8.5](https://www.cisecurity.org/controls/cis-controls-navigator/v8) which advises collecting detailed audit logs for sensitive data. Proper logging is a key component of a robust security posture.

Let me know if you have any other questions! Happy logging!