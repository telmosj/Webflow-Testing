# Ensure MFA Delete is enabled on S3 buckets

## Summary
Using Multi-Factor Authentication (MFA) delete on your important S3 buckets is a smart way to add an extra layer of security. It requires users to provide a second form of authentication when changing bucket versioning or deleting object versions. While it takes a bit of extra work to set up, it can go a long way in protecting your data if your credentials are ever compromised.

## Where did this come from?
This recommendation comes from the CIS Amazon Web Services Foundations Benchmark v3.0.0 released on 01-31-2024. You can download a copy of the full benchmark from the [CIS website](https://downloads.cisecurity.org/#/). For more details, check out the [AWS docs on MFA delete](https://docs.aws.amazon.com/AmazonS3/latest/dev/UsingMFADelete.html) and this helpful [AWS Security Blog post](https://aws.amazon.com/blogs/security/securing-access-to-aws-using-mfa-part-3/).

## Who should care?
This is most relevant for:
- S3 administrators responsible for securing sensitive data 
- Security engineers looking to strengthen the AWS environment
- Compliance officers ensuring alignment with security benchmarks

## What is the risk?
The main risk is unauthorized deletion or modification of important data in S3. If an attacker gains access to valid credentials, they could wreak havoc by deleting entire buckets or modifying object versions. MFA delete helps mitigate this risk by requiring a second factor to make those destructive changes. It's not foolproof, but it makes life harder for attackers.

## What's the care factor? 
You should strongly consider enabling MFA delete for any S3 buckets containing sensitive, regulated, or mission-critical data. Sure, it adds friction to the bucket management process, but that friction is worth it for your most important assets. For run-of-the-mill buckets, you can probably get away without it. Use your judgment based on the value of the data.

## When is it relevant?
MFA delete is a good idea when:
- You have sensitive data in S3 that needs extra protection 
- Compliance standards like PCI, HIPAA, GDPR, etc apply to your buckets
- Multiple users/roles have permissions to modify and delete buckets

It may be overkill if:
- The bucket only contains public or non-sensitive data
- Access is already tightly restricted using other controls 
- Bucket modification is fully automated by trusted processes

## What are the trade-offs?
Enabling MFA delete does come with some downsides:
- Administrators need to have MFA devices and use them every time 
- Bucket changes take a bit longer with the extra authentication step
- May break automation scripts that aren't MFA-aware
- Some additional setup and configuration required

So it's not something to enable lightly everywhere. Weigh the security benefits against the administrative overhead for each use case.

## How to make it happen?
Here's how to enable MFA delete on an S3 bucket:

1. Use your root account credentials. MFA delete can only be enabled by the root user.

2. Ensure you have an MFA device configured for your root account. See [the docs](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_mfa_lost-or- broken.html) if you need to set one up.

3. Install and configure the AWS CLI if you haven't already. MFA delete can't currently be enabled via the console. 

4. Run this command to enable versioning and MFA delete on your bucket:

```
aws s3api put-bucket-versioning --profile my-root-profile --bucket Bucket_Name --versioning-configuration Status=Enabled,MFADelete=Enabled --mfa "arn:aws:iam::aws_account_id:mfa/root-account-mfa-device passcode"
```

Replace `my-root-profile`, `Bucket_Name`, `aws_account_id`, `root-account-mfa-device`, and `passcode` with your actual values.

5. Verify the settings with:

```
aws s3api get-bucket-versioning --bucket Bucket_Name
```

Look for `<MfaDelete>Enabled</MfaDelete>` in the output. 

That's it! Your bucket is now protected with MFA delete. 

## What are some gotchas?
A few things to watch out for:

- MFA delete can only be enabled by the root account, not IAM users
- Versioning must also be enabled on the bucket
- The MFA device must be physical (virtual devices aren't allowed)
- Permissions to change bucket versioning require `s3:PutBucketVersioning`

See the [AWS docs](https://docs.aws.amazon.com/AmazonS3/latest/dev/UsingMFADelete.html) for full details.

## What are the alternatives?
Some other options to control deletion of objects in S3:

- Restrict `s3:DeleteObject` permission on sensitive buckets
- Enable Versioning to preserve deleted objects
- Enable [S3 Object Lock](https://docs.aws.amazon.com/AmazonS3/latest/dev/object-lock.html) to prevent deletion for compliance
- Monitor and alert on object deletions using tools like CloudTrail and CloudWatch

Each has pros and cons. Using multiple tactics in combination with MFA delete is the most robust approach.

## Explore further
- Brush up on the CIS Benchmark recommendations for S3 in [section 2.1](https://d1.awsstatic.com/whitepapers/compliance/AWS_CIS_Foundations_Benchmark.pdf) 
- Learn how to [recover deleted S3 objects](https://docs.aws.amazon.com/AmazonS3/latest/user-guide/undelete-objects.html)
- See how to use MFA delete with the [S3 APIs](https://docs.aws.amazon.com/AmazonS3/latest/API/API_DeleteObject.html)

Hopefully this gives you a solid understanding of why and how to enable MFA delete on your important S3 buckets! Let me know if you have any other questions.