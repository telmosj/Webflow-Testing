# 4.8 Ensure S3 bucket policy changes are monitored (Manual)

## Summary
It's critical to keep a close eye on your S3 bucket policies to ensure they aren't overly permissive. One powerful way to do this is to set up real-time monitoring of S3 API calls using CloudWatch Logs and metric filters. This allows you to quickly detect and respond to any unexpected changes to your S3 bucket policies.

## Where did this come from?
This recommendation comes from the CIS Amazon Web Services Foundations Benchmark v3.0.0 - 01-31-2024. You can find more details and download a copy of the full benchmark from the [CIS website](https://downloads.cisecurity.org/#/). 

The AWS documentation also provides helpful guides on [monitoring CloudTrail events with CloudWatch alarms](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudwatch-alarms-for-cloudtrail.html) and [receiving CloudTrail log files from multiple regions](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/receive-cloudtrail-log-files-from-multiple-regions.html).

## Who should care?
This matters for:
- Cloud security engineers responsible for securing AWS environments
- DevOps teams who manage S3 storage and need to maintain least privilege access 
- Compliance officers who must ensure S3 policies meet regulatory requirements
- IT leaders and CISO who oversee the organization's cloud security posture

## What is the risk?
Overly permissive S3 bucket policies dramatically increase the risk of sensitive data exposure and compliance violations. Without proper monitoring, an attacker could surreptitiously modify bucket policies to open up access. This could allow them to steal private data, inject malware, or use your S3 buckets for illegal hosting.

Even accidental policy changes by admins could expose private files to the internet. There have been numerous high-profile data leaks caused by misconfigured S3 buckets. Real-time alerting on policy modifications is crucial to preventing these types of incidents.

## What's the care factor?
Security teams should treat monitoring of S3 bucket policies as a top priority. S3 buckets are a common target for attackers, since they often contain an organization's most sensitive data. A single leak from a misconfigured bucket could lead to massive reputation damage, financial losses, and regulatory penalties.  

While the technical work to set up the monitoring isn't too difficult, it's the type of fundamental security control that requires consistent effort and diligence. Taking the time to properly implement this CIS recommendation is well worth it to prevent catastrophic S3 misconfigurations.

## When is it relevant?
This control is relevant for virtually all AWS deployments that use S3 storage, which is the vast majority. The only exceptions would be environments that use S3 solely for public, non-sensitive data. Even then, monitoring bucket policies is still a good practice to detect any changes that could indicate a breach.

One scenario where you may want to make exceptions is for CI/CD pipelines or infrastructure-as-code processes that automatically provision S3 buckets. Those automated policy changes are probably fine, but be sure to still monitor them closely.

## What are the trade-offs?
The main cost of this control is the time needed to configure the CloudWatch metric filters and alarms. It's not a ton of work, but it does require some ongoing tuning and monitoring to ensure the alerts are firing properly. 

You also have to factor in the operational overhead of responding to the alerts. Expect some false positives that will need to be investigated, especially when first implementing the control. Over time you can dial in the alerting to minimize unnecessary notifications.

## How to make it happen?
Here are the detailed steps to implement S3 bucket policy monitoring with CloudTrail and CloudWatch:

1. Ensure you have at least one active multi-region CloudTrail trail:
```
aws cloudtrail describe-trails
```
Look for `"IsMultiRegionTrail": true`

2. Confirm the trail is logging to a CloudWatch Logs log group: 
```
aws cloudtrail get-trail-status --name <trail-name>
```
Look for `"IsLogging": true`

3. Ensure the trail is capturing all management events:
```
aws cloudtrail get-event-selectors --trail-name <trail-name>
```
Verify there is an event selector with `"IncludeManagementEvents": true` and `"ReadWriteType": "All"`

4. Create a CloudWatch Logs metric filter for S3 bucket policy changes:
```
aws logs put-metric-filter \
  --log-group-name <cloudtrail_log_group_name> \ 
  --filter-name S3BucketPolicyChanges \
  --metric-transformations \
      metricName=S3BucketPolicyChanges,metricNamespace=CISBenchmark,metricValue=1 \
  --filter-pattern '{ ($.eventSource = s3.amazonaws.com) && (($.eventName = PutBucketAcl) || ($.eventName = PutBucketPolicy) || ($.eventName = PutBucketCors) || ($.eventName = PutBucketLifecycle) || ($.eventName = PutBucketReplication) || ($.eventName = DeleteBucketPolicy) || ($.eventName = DeleteBucketCors) || ($.eventName = DeleteBucketLifecycle) || ($.eventName = DeleteBucketReplication)) }'
```

5. Create an SNS topic for the alarm notifications:
```  
aws sns create-topic --name CIS-S3-Bucket-Policy-Changes
```

6. Subscribe your email to the SNS topic:
```
aws sns subscribe \
  --topic-arn <sns_topic_arn> \
  --protocol email \
  --notification-endpoint <your_email@example.com>
```

7. Create the CloudWatch alarm:
```
aws cloudwatch put-metric-alarm \
  --alarm-name CIS-4.8-S3-Bucket-Policy-Changes \
  --metric-name S3BucketPolicyChanges \
  --statistic Sum \
  --period 300 \
  --threshold 1 \
  --comparison-operator GreaterThanOrEqualToThreshold \
  --evaluation-periods 1 \
  --namespace 'CISBenchmark' \
  --alarm-actions <sns_topic_arn>  
```

## What are some gotchas?
The CloudTrail trail must have permissions to create and write to the CloudWatch Logs log group. Ensure the trail is configured with a role that has the necessary `logs:CreateLogStream` and `logs:PutLogEvents` permissions.

The SNS topic policy also needs to allow CloudWatch to publish to it. When you create the alarm, CloudWatch will automatically update the topic policy if needed. Just be aware in case you run into SNS permission errors later.

## What are the alternatives?
Instead of using CloudWatch Logs and metric filters, you could stream the CloudTrail events to a 3rd party SIEM for monitoring and alerting. Most SIEM platforms have built-in support for ingesting CloudTrail logs and analyzing them for suspicious activity.

Bucket policies aren't the only way to control access to S3. You can also use IAM policies and S3 Access Control Lists (ACLs) to govern permissions. It's a good idea to monitor for changes in those as well, which you can do through a similar CloudTrail + CloudWatch Logs approach.

## Explore further
- CIS AWS Foundations Benchmark 1.2.0: https://www.cisecurity.org/benchmark/amazon_web_services/  
- AWS User Guide - Monitoring CloudTrail with CloudWatch Logs: https://docs.aws.amazon.com/awscloudtrail/latest/userguide/monitor-cloudtrail-log-files-with-cloudwatch-logs.html
- AWS User Guide - Creating an S3 Bucket Policy: https://docs.aws.amazon.com/AmazonS3/latest/userguide/example-bucket-policies.html
- CIS AWS Foundations Benchmark 2.1.1 - S3 Buckets Recommendations: https://docs.aws.amazon.com/securityhub/latest/userguide/securityhub-cis-controls.html#cis-2.1.1

The CIS AWS Foundations Benchmark provides several other recommendations related to securing S3, such as enabling MFA delete (2.1.2), blocking public access (2.1.4), etc.