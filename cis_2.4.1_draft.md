# 2.4.1 Ensure that encryption is enabled for EFS file systems (Automated)

## Summary
AWS Elastic File System (EFS) provides a simple and scalable way to store and access files in the cloud. However, it's important to ensure that your sensitive data stored in EFS is properly secured. One key measure is enabling encryption at rest for all your EFS file systems to protect the confidentiality of your data. 

## Where did this come from?
This recommendation comes from the CIS Amazon Web Services Foundations Benchmark v3.0.0 - 01-31-2024. You can download the full benchmark from the [CIS website](https://downloads.cisecurity.org/#/).

For more information on EFS encryption, refer to the [AWS documentation](https://docs.aws.amazon.com/efs/latest/ug/encryption-at-rest.html).

## Who should care? 
This recommendation is relevant for:
- Security engineers responsible for ensuring data is protected in the cloud
- Compliance officers who need to meet regulatory and security standards 
- Developers and operations teams who work with sensitive data stored in EFS

## What is the risk?
Leaving your EFS file systems unencrypted exposes the data to unauthorized access if the underlying storage is ever compromised. A malicious actor who gains access to the physical storage could potentially read all the data in plaintext.

Encrypting data at rest is an important layer of defense-in-depth. It significantly reduces the blast radius of a data breach from storage layer compromises. Encryption ensures data remains confidential and unusable without access to the encryption keys.

## What's the care factor?
For organizations dealing with sensitive, regulated, or mission-critical data, encryption at rest should be a top priority and a hard requirement for EFS usage. The impact of sensitive data exposure can be severe - ranging from reputational damage, loss of intellectual property, to heavy regulatory fines.

For other less sensitive workloads, encryption at rest is still a best practice and relatively low-hanging fruit to implement, providing a strong additional layer of security. 

## When is it relevant?
Encrypting EFS file systems is relevant when:
- Storing any sensitive, private, or regulated data, such as PII, PHI, financial data
- Subject to compliance regimes like HIPAA, PCI-DSS, GDPR etc. that mandate encryption
- Required by organization's security policies

Encryption may be less critical for:
- Publicly accessible data that is not sensitive if disclosed
- Temporary or ephemeral workloads where data has very short lifespan
- Performance-sensitive workloads where the encryption overhead is unacceptable (evaluate carefully)

## What are the trade offs?
Enabling encryption for EFS does have some costs and considerations:

- KMS costs: EFS encryption uses AWS Key Management Service (KMS) keys which incurs additional charges based on API requests.
- Cannot be enabled after creation: Encryption must be enabled when the EFS file system is first created. You cannot enable it later.
- Small performance overhead: Encryption adds a small latency overhead. For most workloads this is negligible but important to test.
- Snapshot limitations: Some EFS snapshot features are not supported with encryption enabled. 
- Recovery complexity: Encryption adds extra steps to backup and recovery processes. Ensure encryption keys are backed up.

## How to make it happen?
Encryption can be enabled when creating a new EFS file system through the AWS Management Console:

1. Open the Amazon EFS console at https://console.aws.amazon.com/efs/
2. Click "Create file system"
3. Configure the VPC, mount targets, and tags 
4. Under "Configure optional settings", check the "Enable encryption" checkbox
5. Choose "aws/elasticfilesystem" (default) or a customer managed CMK from the "Select a KMS master key" dropdown
6. Click "Next" to review and then "Create File System"

To create an encrypted EFS file system using the AWS CLI:

1. Create a unique creation token (e.g. randomly generate UUID)

```
creation_token=$(uuidgen)
```

2. Create the encrypted EFS file system

```
aws efs create-file-system \
--creation-token $creation_token \
--encrypted \
--kms-key-id <key-id>  # Optional - default is "aws/elasticfilesystem"
```

3. Note the `FileSystemId` from the response
4. Create mount targets in desired subnets

```
aws efs create-mount-target \
--file-system-id <FileSystemId> \
--subnet-id <SubnetId>
```

5. Repeat for each subnet/AZ 
6. Mount the EFS file system on your EC2 instances

For existing unencrypted file systems, you must create a new encrypted file system and migrate data:

1. Create a new encrypted EFS file system and mount targets (as shown above)
2. Mount both old and new file systems on an EC2 instance 
3. Copy data from the old file system to the new encrypted one using rsync or similar
4. Update applications to use the new file system 
5. Verify application functionality
6. Delete the old unencrypted file system

Repeat this process for all EFS file systems in all regions.

## What are some gotchas?
Some things to watch out for when enabling EFS encryption:

- EFS encryption uses KMS, so you must ensure you have the necessary IAM permissions to use the desired KMS key. The IAM user must have `kms:Encrypt`, `kms:Decrypt`, `kms:ReEncrypt*`, `kms:GenerateDataKey*`, and `kms:DescribeKey` on the key. 
- When using EC2 instance profile, ensure it includes `kms:Decrypt` permission on the KMS key so the EC2 instance can mount the encrypted file system.
- Some EFS features like Fast Snapshot Restore (FSR) are not supported on encrypted file systems. 
- When using a custom KMS key, ensure it is in the same region as your EFS

See the [AWS KMS Developer Guide](https://docs.aws.amazon.com/kms/latest/developerguide/overview.html) for more info on working with KMS.

## What are the alternatives?
The main alternative to using EFS with encryption enabled is to use a different storage service that provides server-side encryption by default, such as:

- Amazon S3 with default encryption enabled 
- Amazon EBS volumes with default encryption enabled

If you wish to continue using EFS but have concerns about the performance impact of encryption, you could explore using client-side encryption to encrypt data before storing it on EFS. This moves the encryption overhead to the client.

## Explore further
- [AWS Identity and Access Management for EFS docs](https://docs.aws.amazon.com/efs/latest/ug/auth-and-access-control.html)
- [AWS KMS Developer Guide](https://docs.aws.amazon.com/kms/latest/developerguide/overview.html)
- Relevant CIS AWS benchmarks:
   - Section 2 - Storage
   - Section 3 - Logging
   - Section 4 - Monitoring