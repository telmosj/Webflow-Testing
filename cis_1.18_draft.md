# 1.18 Ensure IAM instance roles are used for AWS resource access from instances (Automated)

## Summary
When accessing AWS resources from EC2 instances, it's important to use IAM roles rather than hard-coding AWS access keys. Using IAM roles is more secure since the permissions can be easily managed and rotated without needing to update the EC2 instances themselves. It's a best practice recommended by AWS themselves. 

## Where did this come from?
This recommendation comes from the CIS Amazon Web Services Foundations Benchmark v3.0.0 released on 01-31-2024. You can download the full benchmark from the [CIS website](https://downloads.cisecurity.org/#/). The benchmark provides prescriptive guidance for configuring security options for a subset of Amazon Web Services with an emphasis on foundational, testable, and architecture agnostic settings.

For more background, check out the [AWS documentation on IAM roles for EC2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html) and [how to use IAM roles with EC2 instances](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_switch-role-ec2.html).

## Who should care?
This is most relevant for:
- AWS administrators responsible for securing EC2 instances 
- Developers launching EC2 instances that need to access other AWS services
- Security engineers looking to enforce best practices across an AWS environment
- Compliance officers ensuring the use of IAM is audited

## What is the risk?
The main risk of not using IAM roles is that AWS access keys get encoded directly into application code or configuration files on the EC2 instances. This is problematic for a few reasons:

1. If those keys are compromised, they can be used from outside of AWS to access resources. An attacker wouldn't need ongoing access to the EC2 instance.  

2. When keys are hard-coded, they tend to not get rotated properly due to the risk of breaking applications. Over time, more and more people who no longer work at the company may know the keys.

3. Updating the permissions required the application code or AMI to be updated on each EC2 instance. This is time consuming and easy to miss places resulting in inconsistent permissions.

Using IAM roles, if the permissions an application needs change, or a key is suspected to be compromised, the IAM role can be updated centrally without touching the individual EC2 instances. This reduces the risk of unauthorized access and makes managing access much simpler.

## What's the care factor?
For companies that use EC2 heavily, this should be a high priority item to get right. Setting up IAM roles is pretty simple, but refactoring applications later to remove hard-coded keys can be a big effort. It's best to start using IAM roles as early as possible and make it the standard pattern.

For smaller deployments with just a few instances that rarely change, and where the AWS access keys are properly managed, using IAM roles is still a good best practice but may be lower priority than other security initiatives. 

## When is it relevant?
You should use IAM roles when:
- You are deploying an application or script on an EC2 instance that needs access to make AWS API calls to access other services like S3, RDS, DynamoDB etc.
- You are using configuration management tools like Chef, Puppet, Ansible etc. to configure EC2 instances that require access keys to be set
- You are using the AWS CLI or SDKs on the EC2 instance to make API calls

You may not need IAM roles when:
- Your EC2 instance does not make any AWS API calls. For example a simple web server that has no backend AWS dependencies.
- You are running a 3rd party application that does not have built in support for IAM roles. Although even in this case, custom credential provider chains can often be configured to use IAM roles.

## What are the trade offs?
The main benefit of using IAM roles is increased security and simplified permissions management as detailed above. 

The downsides are:
- For existing applications, updating to IAM roles requires some development effort to refactor how credentials are handled. This may be significant for complex applications.
- IAM roles are not supported by all AWS services and 3rd party applications, so depending on your use case it may not be feasible (although this is increasingly rare).
- Using roles can make it harder to test applications locally since the EC2 metadata API is not available outside of an EC2 instance. It may require a code change to use different credential sources in dev vs prod.

## How to make it happen?
Switching to IAM roles requires two steps:
1. Create an IAM role with the required permissions 
2. Associate that role with an EC2 instance

To create the IAM role:

1. Open the IAM console at https://console.aws.amazon.com/iam/.
2. In the navigation pane, click Roles, and then click Create role.
3. Click EC2 from the list of services, and click Next: Permissions.
4. On the Attach permissions policies page, select the IAM policies to attach to the role. For example, if the application on the instance needs full access to S3 and read-only access to DynamoDB, select `AmazonS3FullAccess` and `AmazonDynamoDBReadOnlyAccess`.
5. Click Next: Tags.
6. (Optional) Add metadata to the role by attaching tags as key-value pairs. 
7. Click Next: Review.
8. Enter a name for the role, such as `MyEC2S3DynamoDBRole`, and click Create role.

Then to attach the role to an EC2 instance:

1. Open the Amazon EC2 console at https://console.aws.amazon.com/ec2/.
2. In the navigation pane, click Instances.
3. Select the instance, click Actions, Instance Settings, Attach/Replace IAM Role.
4. Select the IAM role you just created from the drop down and click Apply.

The EC2 instance will now be able to access AWS APIs using the permissions of the IAM role, without needing any hard-coded access keys.

For an automated approach, you can specify an IAM role when launching an instance using the AWS CLI `run-instances` command. Specify the `--iam-instance-profile` parameter with the IAM instance profile (role) name.

You can also attach IAM roles to Auto Scaling Groups by specifying the IAM instance profile in the launch configuration.

## What are some gotchas?
A few things to keep in mind:
- For an application to use the IAM role, it must be coded to get temporary credentials from the EC2 instance metadata service at http://169.254.169.254. The AWS SDKs handle this transparently, but for custom applications, reference the AWS documentation for how to retrieve and use the role credentials.
- IAM roles are not supported on t1.micro instances.
- By default, IAM roles cannot be attached to an instance after it is launched. You must stop and start the instance for the role change to take effect.
- The IAM role must include permissions required for all AWS services used by the applications on the instance. Be specific and limit permissions to only what is absolutely required. For example, instead of `AmazonS3FullAccess`, specify a custom policy with access to specific buckets and actions needed.
- Be careful not to mix usage of IAM roles and access keys on the same instance to avoid confusion. 

## What are the alternatives?
The main alternative is to use hard-coded access keys on the EC2 instance, but for the reasons discussed above this is not recommended.

If you need to provide AWS access to an application running outside of AWS, you can use IAM user access keys but these should be rotated frequently. A better approach is to use an external identity provider like Active Directory, and federate access to AWS using [SAML](https://aws.amazon.com/blogs/security/how-to-implement-a-general-solution-for-federated-apicli-access-using-saml-2-0). 

## Explore further
- For more details on IAM roles, read the [AWS Documentation](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html).
- For a hands-on lab to get started with IAM roles, try this [qwikLAB](https://amazon.qwiklabs.com/focuses/308?parent=catalog).
- For a broader discussion of AWS access management best practices, see the AWS whitepaper [IAM Best Practices](https://d1.awsstatic.com/whitepapers/Security/AWS_Security_Best_Practices