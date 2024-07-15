# 3.5 Ensure CloudTrail logs are encrypted at rest using KMS CMKs (Automated)

## Summary
AWS CloudTrail is a super handy service that records API calls in your AWS account and makes the logs available for you to review. To add an extra layer of protection to your CloudTrail logs, it's a smart move to encrypt them at rest using AWS Key Management Service (KMS) customer master keys (CMKs). This ensures only authorized users with the right permissions can access the sensitive info in your audit logs.

## Where did this come from?
This recommendation comes straight from the "CIS Amazon Web Services Foundations Benchmark v3.0.0 - 01-31-2024". You can download the full benchmark document from https://downloads.cisecurity.org/#/ for all the juicy details. The AWS docs also have great info on encrypting CloudTrail logs with KMS here: https://docs.aws.amazon.com/awscloudtrail/latest/userguide/encrypting-cloudtrail-log-files-with-aws-kms.html

## Who should care? 
A few key personas who should pay attention to this:

- Security analysts with responsibility for monitoring and protecting AWS environments
- Compliance officers with need to ensure audit logs are secured and only accessible to authorized personnel 
- DevOps engineers with ownership of configuring logging and security controls in AWS accounts

## What is the risk?
If you leave your CloudTrail logs unencrypted, you run the risk of:

- Unauthorized access to sensitive API activity data if someone gains access to the S3 bucket storing the logs
- Failing compliance audits that require encryption of log data at rest
- The logs getting accidentally or maliciously deleted without the additional protection encryption provides

Encrypting with KMS CMKs significantly reduces these risks by requiring specific decrypt permissions in addition to S3 bucket access permissions to be able to read the log contents. The CMKs also give you more granular control and monitoring capabilities over the encryption keys.

## What's the care factor?
For most organizations, encrypting CloudTrail logs with KMS CMKs should be a high priority, especially if you:

- Store sensitive data in AWS or use it to run critical workloads
- Need to comply with security standards like PCI, HIPAA, SOC, ISO, etc.
- Want defense in depth for your audit log data

The added protection is well worth the small amount of extra effort and minimal cost. Logs are a crucial data source for investigating incidents, so it's important the integrity and confidentiality of that data is preserved.

## When is it relevant?
Encrypting CloudTrail logs at rest is relevant any time you have CloudTrail enabled and are storing the logs in S3 (which is most of the time). A few specific situations:

- When you first set up CloudTrail in an account
- If you are updating your CloudTrail configuration 
- During periodic security/compliance reviews

The only situation where it may not be relevant is if you are streaming CloudTrail events to a 3rd party SIEM in real-time and then discarding the logs. But even then, best practice is to keep a copy in S3 for retention and having that copy encrypted is still a good idea.

## What are the trade-offs?
The main trade-offs/costs to consider are:

- Small cost increase (KMS charges $1/month per CMK and a small fee per 10,000 encryption operations)
- Additional set up time to create the CMK, assign permissions, and configure CloudTrail to use it (though this is pretty quick, especially if using IaC)
- If you automate key rotation, there will be some additional operational overhead to manage that (but it's a security best practice you should be doing anyway)

Overall the costs are minimal compared to the security and compliance benefits in most cases. The only other thing to keep in mind is if you use a 3rd party SIEM or log analysis tool, you'll need to ensure it supports decrypting KMS-encrypted data.

## How to make it happen?
Here's a step-by-step guide to enabling CloudTrail log encryption with KMS:

1. Create a new symmetric encryption KMS CMK or choose an existing one in the same region as your S3 bucket 
2. Update the key policy on the CMK as follows:
- Add a statement allowing CloudTrail to describe the key:

```json
{
  "Sid": "Allow CloudTrail access",
  "Effect": "Allow",  
  "Principal": {
    "Service": "cloudtrail.amazonaws.com"    
  },
  "Action": "kms:DescribeKey",
  "Resource": "*"
}
```

- Add a statement allowing CloudTrail to use the key to encrypt log files:

```json
{
  "Sid": "Allow CloudTrail to encrypt logs",  
  "Effect": "Allow",
  "Principal": {
     "Service": "cloudtrail.amazonaws.com"    
  },
  "Action": "kms:GenerateDataKey*",
  "Resource": "*",  
  "Condition": {
    "StringLike": {
      "kms:EncryptionContext:aws:cloudtrail:arn": [
        "arn:aws:cloudtrail:*:aws-account-id:trail/*"
      ]
    }   
  }
}
```

- Add a statement allowing your IAM users/roles to decrypt log files:

```json
{
  "Sid": "Enable CloudTrail log decrypt permissions",
  "Effect": "Allow",
  "Principal": {
    "AWS": "arn:aws:iam::aws-account-id:user/username" 
  },
  "Action": "kms:Decrypt", 
  "Resource": "*",
  "Condition": {
    "Null": {
      "kms:EncryptionContext:aws:cloudtrail:arn": "false"  
    }
  }
}
```

3. Open the CloudTrail console and edit your Trail 
4. In the S3 section, enable "Encrypt log files" and choose the CMK you created/updated in step 1
5. Save the Trail changes

Your logs will now be encrypted using SSE-KMS with the specified CMK! 🎉

## What are some gotchas?
A few things to watch out for:

- The CMK must be in the same region as the S3 bucket used by CloudTrail
- Users will need `kms:Decrypt` permission (as shown in step 2 above) to be able to read the encrypted log files
- If you don't use the AWS CLI profiles that come with the decrypt permission, remember to specify `--no-verify-ssl` when accessing the API or add the appropriate CA cert
- There is no way to change the CMK on an existing Trail. If you need to rotate keys, you'll have to create a new Trail and CMK.
- If your CMK is scheduled for deletion, CloudTrail will fall back to encrypting with SSE-S3, so make sure you don't accidentally delete keys that are in use

## What are the alternatives?
The main alternative is to use AES-256 server-side encryption (SSE-S3) instead of a CMK, which is enabled by default for CloudTrail. This still encrypts your logs at rest, but uses an Amazon S3-managed key rather than a key in your AWS KMS account. The benefits of a CMK are:

- Added level of control and granularity for permissions
- Centralized auditing and monitoring through CloudTrail's KMS event logs 
- Ability to use an external key store for even greater control, if needed

But SSE-S3 still provides robust encryption and may be sufficient for some use cases. The other option is to use client-side encryption before the logs even reach S3, but this adds a lot of complexity vs. just checking a box in the CloudTrail config.

## Explore further
For more details and discussion of CloudTrail encryption, log security best practices, and AWS KMS, check out:

- AWS CloudTrail User Guide - Encrypting CloudTrail Log Files with AWS KMS: https://docs.aws.amazon.com/awscloudtrail/latest/userguide/encrypting-cloudtrail-log-files-with-aws-kms.html 
- AWS Key Management Service Developer Guide: https://docs.aws.amazon.com/kms/latest/developerguide/overview.html
- CIS AWS Foundations Benchmark - All recommendations related to logging and monitoring: https://www.cisecurity.org/benchmark/amazon_web_services/
