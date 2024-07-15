# Ensure S3 bucket access logging is enabled on the CloudTrail S3 bucket (Automated)

Want to keep an eye on who's poking around in your CloudTrail S3 bucket? Enable access logging and you'll have a handy record of every request made to the bucket. It's like having a security camera watching your digital valuables.

## Where did this come from?
This recommendation comes straight from the "CIS Amazon Web Services Foundations Benchmark v3.0.0 - 01-31-2024". You can download the full benchmark from the [CIS website](https://downloads.cisecurity.org/#/). The benchmark provides tons of great tips for locking down your AWS environment. For more details on S3 bucket logging specifically, check out the [AWS documentation](https://docs.aws.amazon.com/AmazonS3/latest/dev/ServerLogs.html).

## Who should care? 
- Security analysts with a need to monitor access to sensitive log data
- Compliance officers with a responsibility to maintain audit records
- Paranoid system administrators with an obsession for tracking everything

## What is the risk?
Without access logging enabled, you'll have no idea who is accessing your CloudTrail S3 bucket and what they are doing in there. This blind spot could allow an attacker to steal or tamper with your sensitive log files without raising any alarms. 

While logging can't directly prevent unauthorized access, it serves as an essential detection control. Proper logging can help you identify suspicious behavior, understand the scope of a breach, and provide forensic evidence during investigations.

## What's the care factor?
If your CloudTrail S3 bucket contains highly sensitive data, you should definitely care about enabling access logging. Logs are often a goldmine for attackers since they can reveal juicy info about your environment. Treat your logs like the crown jewels.

For less sensitive log buckets, logging may be a lower priority but is still a fundamental security best practice that every organization should follow. Don't underestimate the value of your log data.

## When is it relevant?
Access logging should be enabled for any S3 bucket, but it is especially critical for buckets storing sensitive log data including:

- CloudTrail logs
- Application logs containing PII or financial data 
- Audit logs required for compliance

In rare cases where logs contain no sensitive info and are accessed very infrequently, you may determine logging is not worth the added storage costs and management overhead. But when in doubt, log it out.

## What are the trade offs?
The main downside to access logging is the cost of storing all those log files. The more requests made to your bucket, the more logs you'll rack up. But storage is cheap, and you can always transition older logs to a cheaper storage class like Glacier.

Access logs can also be noisy with lots of routine entries that you'll need to sift through. With logging enabled, you'll likely want to invest in a log management tool to help filter, search, and make sense of it all.

## How to make it happen?
Ready to get logging? Here's how:

From the web console:
1. Sign into the [AWS Management Console](https://console.aws.amazon.com/s3) and open the S3 dashboard 
2. Click on the bucket you want to enable logging for
3. Open the **Properties** tab
4. Under **Server access logging** click **Edit**
5. Check the **Enable** box 
6. Choose the **Target bucket** where you want the logs delivered (can be the same or different bucket)
7. Specify a **Target prefix** to group the logs (optional but recommended)
8. Click **Save changes**

From the AWS CLI:
1. Update the following JSON, replacing the placeholders:
```
{
  "LoggingEnabled": {
    "TargetBucket": "<target-bucket-name>",  
    "TargetPrefix": "<log-prefix>/"
  }
}
```
2. Save the JSON in a file called `logging.json`
3. Run this command to enable logging:
```
aws s3api put-bucket-logging --bucket <your-bucket-name> --bucket-logging-status file://logging.json
```

## What are some gotchas?
Before you can enable logging on a bucket, you'll need to make sure the target bucket where the logs will be delivered already exists. The buckets can be in different AWS accounts but must be in the same region.

Your IAM user or role will need the following permissions:
- `s3:GetBucketLogging` 
- `s3:PutBucketLogging`

If the source and target buckets are owned by different AWS accounts, you'll need to grant cross-account permissions. The easiest way is to update the target bucket policy to allow the source account access to write the logs:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",      
      "Principal": {
        "AWS": "arn:aws:iam::<source-account-id>:root"  
      },
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::<target-bucket-name>/<prefix>/*"    
    }
  ]
}
```

For all the hairy details on setting permissions for cross-account logging, study the [AWS docs](https://docs.aws.amazon.com/AmazonS3/latest/dev/enable-logging-programming.html).

Once logging is enabled, it can take a few minutes before the first log files show up in the target bucket. Also keep in mind logging applies only to new requests. You can't retroactively log past requests.

## What are the alternatives?
Instead of using S3's built-in access logs, you could analyze the data from AWS CloudTrail. CloudTrail logs all API activity in your account, including S3 requests. This gives you one centralized place to monitor all the things. 

However, CloudTrail has some limitations compared to S3 access logs:
- CloudTrail only logs at the API level, so you can't see things like HTTP headers or query strings
- There is a slight delay (about 15 min) before events show up in CloudTrail
- For large buckets with millions of objects, enabling CloudTrail data events can get pricey

So while CloudTrail is super handy, it is not a full replacement for direct S3 access logs in all cases.

## Explore further
- AWS User Guide on [Logging requests using server access logging](https://docs.aws.amazon.com/AmazonS3/latest/dev/ServerLogs.html) 
- AWS User Guide on [Enabling CloudTrail event logging for S3 buckets and objects](https://docs.aws.amazon.com/AmazonS3/latest/user-guide/enable-cloudtrail-logging-for-s3.html)
- CIS AWS Foundations Benchmark Recommendation 2.9 - Ensure CloudTrail is enabled in all regions
- Tutorial on [analyzing S3 access logs with Athena](https://aws.amazon.com/blogs/big-data/aws-cloudtrail-and-amazon-athena-dive-deep-to-analyze-security-compliance-and-operational-activity/)