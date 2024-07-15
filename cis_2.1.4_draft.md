# Ensure that S3 Buckets are configured with 'Block public access (bucket settings)' (Automated)

## Summary 
Amazon S3 provides granular controls for managing public access to your S3 buckets and objects. By default, all buckets and objects are created with public access disabled. However, a simple misconfiguration or malicious insider with the right IAM permissions could easily make your sensitive data publicly accessible to the world. To prevent this, it's critical to enable the "Block public access" settings on your S3 buckets.

## Where did this come from?
This recommendation comes from the CIS Amazon Web Services Foundations Benchmark v1.5.0 section 2.1.4. You can download the full benchmark from https://downloads.cisecurity.org/#/ for more details. The corresponding AWS documentation on blocking public access can be found at https://docs.aws.amazon.com/AmazonS3/latest/user-guide/block-public-access.html.

## Who should care? 
This is relevant for:
- AWS administrators responsible for configuring and securing S3 buckets
- Application owners storing any sensitive data in S3 
- Security and compliance officers ensuring data is not exposed publicly
- Developers building apps that access data in S3

## What is the risk?
Leaving S3 buckets public poses many risks:
- Accidental data exposure of PII, PHI, financial data, intellectual property, etc.
- Targeted data breach by malicious actors scanning for public buckets
- Compliance violations like HIPAA, PCI, GDPR, etc.
- Reputational damage and loss of customer trust
- Financial penalties and lawsuits

Properly enabling the Block Public Access settings on S3 buckets completely prevents the bucket and its contents from becoming publicly accessible, greatly mitigating these risks.

## What's the care factor?
For any organization storing sensitive data in S3, this should be a top priority and standard configuration. The risks are too high to ignore.

However, for some use cases deliberately serving public content from S3, like hosting public websites, these settings may need to be disabled on certain buckets. But for everything else, blocking public access should be the rule.

## When is it relevant?
You should enable Block Public Access when:
- Standing up any new S3 buckets
- Storing any non-public data in S3
- Performing security/compliance reviews of your AWS environment 

It's not as relevant when:
- Intentionally serving public web content from S3
- Migrating or backing up on-prem public data stores to S3

But for the vast majority of S3 use cases, Block Public Access should always be enabled.

## What are the tradeoffs?
The main tradeoff is additional configuration and management overhead. You need to ensure the right settings get enabled consistently on every new bucket.

There could also be some disruption if you enable it on existing buckets that were unintentionally public. So careful testing is warranted.

Disabling public access could also cause headaches if you later determine some of that data does need to be publicly accessible. You'll need to make a more deliberate effort to expose it appropriately.

## How to make it happen?
There are two levels where you can enforce blocking public access:

### Account level
1. Open the Amazon S3 console
2. Choose "Block Public Access settings for this account"
3. Check the box for "Block all public access"
4. Save changes

This will disable public access for every bucket in your account.

### Bucket level 
1. Open the S3 console
2. Select the bucket 
3. Go to Permissions > Block public access
4. Check the box for "Block all public access"
5. Save changes

Alternatively, you can enable it with the AWS CLI:

```
aws s3api put-public-access-block \
    --bucket my-bucket \
    --public-access-block-configuration "BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true"
```

Audit your configuration by describing each bucket:

```
aws s3api get-bucket-acl --bucket my-bucket
aws s3api get-bucket-policy --bucket my-bucket  
aws s3api get-bucket-policy-status --bucket my-bucket
aws s3api get-public-access-block --bucket my-bucket
```

Look for `"PublicAccessBlockConfiguration": { "BlockPublicAcls": true, ...}` to verify the settings are enabled.

## What are some gotchas?
To perform the configuration you'll need appropriate IAM permissions:
- `s3:GetBucketPublicAccessBlock`
- `s3:PutBucketPublicAccessBlock` 
- `s3:GetBucketPolicyStatus`
- `s3:GetBucketAcl`

Documentation: https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-control-block-public-access.html 

If you enable blocking at the account level, it overrides the individual bucket settings. So use caution as that could have wide-ranging effects.

Blocking public access does not apply retroactively to objects that were already public. You'll need to update the object ACLs separately.

Also, these settings do not prevent access to the bucket/objects by authenticated IAM users, only anonymous public access.

## What are the alternatives?
The main alternative is to rely on carefully crafted bucket policies and IAM to avoid public exposure. But that's error prone and easy to miss.

You could also use AWS Config rules or S3 Event Notifications to automatically detect and remediate public exposures. But that's reactive rather than preventative.

Ultimately, Block Public Access provides the strongest, simplest defense.

## Explore Further
- Read the S3 User Guide on the "Block Public Access" feature: https://docs.aws.amazon.com/AmazonS3/latest/user-guide/block-public-access.html
- Consider also enforcing HTTPS for S3 transport encryption: CIS 3.5
- Ensure S3 buckets have appropriate access logging enabled: CIS 2.1.1
- Apply a standard, secure bucket policy to enforce access requirements and prevent misconfigurations: CIS 2.1.5