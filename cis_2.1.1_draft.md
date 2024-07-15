# 2.1.1 Ensure S3 Bucket Policy is set to deny HTTP requests (Automated)

## Summary
Hey there! Let's chat about a super important AWS security setting - denying HTTP requests to your S3 buckets. By default, S3 allows both HTTP and HTTPS, but to really lock things down, you need to explicitly deny insecure HTTP access. Trust me, it's worth the effort to keep your data safe!

## Where did this come from?
This recommendation comes straight from the "CIS Amazon Web Services Foundations Benchmark v3.0.0 - 01-31-2024". You can download the full benchmark from the [CIS website](https://downloads.cisecurity.org/#/). The benchmark provides a ton of great AWS security best practices to help keep your cloud environment secure. For more details on S3 bucket policies, check out the [AWS docs](https://docs.aws.amazon.com/AmazonS3/latest/userguide/bucket-policies.html).

## Who should care? 
This matters for any AWS administrator responsible for securing S3 buckets that store sensitive data. It's especially critical for security engineers tasked with meeting compliance requirements around data protection.

## What is the risk?
Allowing HTTP access to your S3 buckets opens the door for attackers to intercept sensitive data in transit. They could eavesdrop on the unencrypted traffic and potentially access private information. Configuring your bucket policy to deny HTTP helps mitigate this risk by ensuring data can only be accessed over a secure HTTPS connection. While it's not a silver bullet, it's an important layer in a defense-in-depth strategy.

## What's the care factor?
On a scale from "meh" to "oh sh!t", this is definitely towards the upper end. Leaking sensitive data can lead to big problems - compliance violations, reputational damage, financial losses. While the likelihood of exploitation depends on your threat model, the impact is almost always high. So it's best to be proactive and lock down those buckets.

## When is it relevant?
Denying HTTP access is a smart move for any S3 bucket, but it's absolutely essential for buckets containing sensitive, regulated, or otherwise valuable data. Think PII, financial records, intellectual property, etc. If it's public data...maybe not as critical. Use your judgment based on the sensitivity of the contents.

## What are the trade-offs?
The good news is this is a pretty low-effort security win. The main "cost" is you need to update links and apps to use HTTPS instead of HTTP when accessing the bucket. And make sure your bucket policy doesn't conflict with other access controls. But that's about it - well worth the improved security posture in most cases.

## How to make it happen?
You've got a few options to deny HTTP access:

From the S3 Console:
1. Open the bucket permissions and go to the "Bucket Policy" tab 
2. Add this statement to the policy, replacing `<bucket_arn>` with your bucket ARN:

```
{
  "Sid":"DenyInsecureTransport",  
  "Effect":"Deny",
  "Principal": "*",
  "Action":"s3:*",  
  "Resource":"<bucket_arn>/*",
  "Condition": {
    "Bool": {
      "aws:SecureTransport":"false"
    }
  }
}
```

From the AWS CLI:
1. Save your current bucket policy locally:
`aws s3api get-bucket-policy --bucket <bucket_name> policy.json`

2. Add the HTTP deny statement from above to the policy file

3. Write the updated policy to the bucket:  
`aws s3api put-bucket-policy --bucket <bucket_name> --policy file://policy.json`

## What are some gotchas?
A few things to watch out for:
- The deny statement needs `s3:*` permissions. Make sure the AWS IAM user/role you use to update the policy has sufficient privileges (`s3:PutBucketPolicy` at a minimum).
- Don't forget the `/*` on the end of the resource ARN. That makes the policy apply to all objects in the bucket.
- If you have conflicting ALLOW statements in your policy, the explicit DENY will always take precedence.

## What are the alternatives?
Denying HTTP access in the bucket policy is the way to go for S3. But in general, you can usually toggle HTTP/HTTPS at the load balancer or web server layer too. Also consider enabling default encryption on the bucket so data is protected at rest as well.

## Explore further
- Read up on [S3 security best practices](https://docs.aws.amazon.com/AmazonS3/latest/userguide/security-best-practices.html) for more tips
- Check out the related CIS Benchmark recommendations:
   - 2.1.2 - Ensure Bucket Policy allows HTTPS requests
   - 2.1.3 - Ensure Bucket Policy restricts public read access 
   - 2.1.4 - Ensure Bucket Policy restricts public write access
- Watch out for overly permissive ACLs too. Bucket policies are evaluated first, but ACLs can still grant unintended access if misconfigured.

That's the scoop on locking down S3 to HTTPS-only! It's a quick win that can make a big difference in keeping your cloud data secure. Stay safe out there!