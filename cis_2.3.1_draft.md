# 2.3.1 Ensure that encryption-at-rest is enabled for RDS Instances (Automated)

## Summary
Hey there database admins, listen up! We all know that databases hold some of our most sensitive and mission-critical data, right? Well, if you're using Amazon RDS instances, you absolutely must enable encryption-at-rest. It's the industry standard way to protect your precious data from prying eyes and sticky fingers. Don't worry, it's not too hard and the performance impact is minimal. Let's dive in!

## Where did this come from?
This recommendation comes straight from the pros at the Center for Internet Security (CIS). Specifically, it's part of the "CIS Amazon Web Services Foundations Benchmark v3.0.0" released on 01-31-2024. You can download the full benchmark document from the CIS website: [https://downloads.cisecurity.org/#/](https://downloads.cisecurity.org/#/)

For more juicy details on RDS encryption, check out these handy AWS docs:
- [Encrypting Amazon RDS Resources](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Overview.Encryption.html)  
- [Selecting the Right Encryption Options for Amazon RDS](https://aws.amazon.com/blogs/database/selecting-the-right-encryption-options-for-amazon-rds-and-amazon-aurora-database-engines/)

## Who should care?
This one goes out to all the:
- Database administrators who want to keep sensitive data safe and sound
- Security engineers responsible for locking down AWS environments
- Compliance officers making sure the org is following best practices and standards
- Developers building apps that store regulated or confidential data in RDS

## What is the risk?
Imagine if an attacker was able to gain unauthorized access to your unencrypted RDS instances and databases. They could steal sensitive customer data, intellectual property, financial records - you name it. It would be a major breach with serious consequences. 

Enabling encryption-at-rest is one of the most effective ways to mitigate this risk. It ensures that all the data stored on the RDS instance's underlying storage, automated backups, read replicas, and snapshots is encrypted using the strong industry-standard AES-256 algorithm. So even if the bad guys get in, all they'll find is indecipherable gibberish. Crisis averted!

## What's the care factor?
For most organizations, this should be a top priority, especially if you have any kind of sensitive, regulated, or valuable data living in RDS. The reputational damage and legal penalties of a data breach can be devastating. 

Sure, nothing is 100% hack-proof, but encryption-at-rest is a huge leap forward in your security posture for a pretty small amount of effort. It's a no-brainer, really. Imagine trying to explain to your boss or customers that their data was stolen because you couldn't be bothered to check a box. Not a good look!

## When is it relevant?
You should use RDS encryption in basically any production environment or anywhere you care about the data. It's especially critical for:
- Databases holding personally identifiable information (PII) 
- Regulated workloads subject to HIPAA, PCI-DSS, GDPR, etc.
- Sensitive internal data like financial records, IP, employee info
- Any databases exposed to the internet

The only situation where you might get away without is for purely ephemeral dev/test instances that don't contain real data. But even then, it's a good habit!

## What are the trade offs?
As always, security comes with some costs, but in the case of RDS encryption, they are pretty minimal:
- There's a tiny performance overhead for the encryption/decryption operations, but it's negligible for most workloads
- You can't convert an existing unencrypted RDS instance to encrypted (but there are fairly easy workarounds)
- Encryption can't be disabled once enabled on an instance
- Some old RDS types like db.t2.micro don't support encryption
- Encrypting and decrypting large databases for restores/imports can take a while

In the grand scheme of things, these are small prices to pay for the massive security benefits.

## How to make it happen?
Okay, ready to lock it down? Here's how to enable RDS encryption, step-by-step:

From the AWS Console:
1. Open the RDS console and go to Databases 
2. Select the instance you want to encrypt
3. If it's unencrypted, you'll need to take a manual snapshot first:
   - Click Actions > Take Snapshot
   - Name it and click Take Snapshot
4. Now make an encrypted copy of the snapshot:
    - Select the new snapshot > Actions > Copy Snapshot  
    - Enter a name for the copy
    - Under Encryption, choose Yes
    - Pick the KMS key to use (AWS managed or your own CMK)
    - Click Copy Snapshot
5. Restore the encrypted snapshot to a new instance:
    - Select the encrypted snapshot copy
    - Click Actions > Restore snapshot
    - Enter a name for the new encrypted instance
    - Configure the other settings as needed
    - Click Restore DB Instance
6. Update your app configs to point at the new encrypted instance endpoint
7. Verify it's working, then delete the old unencrypted instance

You can also script it out with the AWS CLI:

```bash
# List DB instances 
aws rds describe-db-instances --query 'DBInstances[*].DBInstanceIdentifier'

# Create a snapshot of the target instance
aws rds create-db-snapshot --db-snapshot-identifier mysnap --db-instance-identifier mydb

# Copy the snapshot with encryption  
aws rds copy-db-snapshot --source-db-snapshot-identifier mysnap --target-db-snapshot-identifier mysnap-encrypted --kms-key-id alias/aws/rds --copy-tags 

# Restore the encrypted snapshot to a new instance
aws rds restore-db-instance-from-db-snapshot --db-instance-identifier mydb-encrypted --db-snapshot-identifier mysnap-encrypted

# Verify encryption is enabled
aws rds describe-db-instances --db-instance-identifier mydb-encrypted --query 'DBInstances[*].StorageEncrypted'
```

## What are some gotchas?
A few things to watch out for when enabling RDS encryption:
- You need the `rds:CopyDBSnapshot` and `kms:CreateGrant` permissions to make encrypted snapshot copies
- For encrypted cross-region snapshot copies, the source and destination regions must have the same KMS CMK
- Encrypted snapshots and instances can only be shared with accounts that you have authorized to access the KMS key
- The performance overhead is generally 5% or less, but could be higher for I/O intensive workloads
- If you're using a custom KMS key (not AWS managed), make sure it's available and properly configured!

## What are the alternatives?
RDS encryption isn't the only way to protect your data. Some other options include:

- Transparent Data Encryption (TDE) at the database engine level (supported for SQL Server and Oracle)
- Application-layer encryption where your app encrypts data before storing in RDS (more complex)
- Disk-level encryption on EC2 instances using EBS encryption (for self-managed databases)
- Third-party encryption plug-ins or partner solutions

Each has pros and cons, but RDS native encryption-at-rest is the simplest and most universally applicable. 

## Explore further
Want to dive deeper into the wonderful world of RDS security? Check out these resources:

- [RDS Security Guide](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/UsingWithRDS.html)
- [Securing RDS and Aurora Databases](https://aws.amazon.com/blogs/database/securing-amazon-rds-and-aurora-postgresql-database-access-with-iam-authentication/)
- [CIS Control v8 3.11 Encrypt Sensitive Data at Rest - requires encryption for sensitive data on servers and databases](https://www.cisecurity.org/controls/cis-controls-navigator/v8)
- [CIS Control v7 14.8 Encrypt Sensitive Information at Rest - requires a separate encryption mechanism beyond native OS tools](https://www.cisecurity.org/controls/cis-controls-navigator/v7-1)

There you have it - RDS encryption in a nutshell. Now go forth and encrypt! Your data will thank you.