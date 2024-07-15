# 1.17 Ensure a support role has been created to manage incidents with AWS Support (Automated)

## Summary
It's important to designate specific personnel to handle support incidents with AWS to ensure issues get addressed promptly and effectively. AWS provides a robust support system, but someone at your organization needs to be responsible for engaging with them when needed. Creating a dedicated IAM role for this purpose is a best practice to grant the necessary access while adhering to the principle of least privilege.

## Where did this come from?
This recommendation comes from the CIS Amazon Web Services Foundations Benchmark v3.0.0 published on 01-31-2024. The full benchmark document can be downloaded from the [CIS website](https://downloads.cisecurity.org/#/). 

The benchmark provides prescriptive guidance for configuring AWS accounts with security best practices in mind. For more details on managing access to AWS Support, refer to the [AWS Support documentation](https://docs.aws.amazon.com/awssupport/latest/user/getting-started.html).

## Who should care?
- Security teams responsible for ensuring least privilege access control in AWS
- Operations teams who engage with AWS Support to resolve technical issues
- Compliance officers validating adherence to security benchmarks like CIS

## What is the risk?
Failing to designate a limited support role can lead to:
- Slower incident response times if the right people don't have access to AWS Support
- Potential privilege escalation if users have excessive permissions beyond just support needs
- Difficulty investigating an incident if no paper trail exists for support engagement

Implementing this recommendation provides an authorized and auditable channel for engaging AWS Support while limiting access as much as possible.

## What's the care factor?
Orgs subject to compliance standards like PCI, HIPAA, SOC, etc should treat this as a high priority item to maintain their security posture. Even if not under compliance pressures, formalizing support engagement is still good practice and should be prioritized as resources allow.

Small teams wearing multiple hats might accept the risk for simplicity, but ideally someone is always explicitly assigned to handle support needs.

## When is it relevant?
Creation of a dedicated support IAM role is most applicable for:
- Larger organizations with multiple teams needing periodic AWS Support 
- Highly regulated industries with strict access control and audit requirements
- Teams practicing mature DevOps with clear separation of duties

It may be overkill for smaller organizations or solo practitioners who just call AWS Support on an as-needed basis with their regular admin account.

## What are the trade-offs?
- Additional time and effort is needed to create and manage a separate IAM role
- Ongoing overhead to ensure the role is assigned to the right people over time
- Potential process bottlenecks if the role is too narrowly scoped
- Frustration for support personnel who now have to switch roles when engaging AWS

## How to make it happen?
1. Create an IAM trust policy allowing specific user(s) to assume the support role:

```json
{
  "Version": "2012-10-17", 
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "<iam_user_arn>"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

2. Create a new IAM role for AWS support:

`aws iam create-role --role-name AWSSupportRole --assume-role-policy-document file://trust-policy.json`

3. Attach the AWS managed "AWSSupportAccess" policy to the new role:

`aws iam attach-role-policy --policy-arn arn:aws:iam::aws:policy/AWSSupportAccess --role-name AWSSupportRole`

4. Assign the role to the users responsible for incident management with AWS Support.

## What are some gotchas?
- The trust policy must specify the full ARN for the IAM user allowed to assume the support role
- AWSSupportAccess is a global AWS managed policy with a fixed ARN that can't be modified
- Inline policies attached to the support role will be additive to the permissions from AWSSupportAccess
- Carefully consider which user(s) have access to assume the support role to maintain separation of duties

## What are the alternatives?
Rather than using the AWS managed policy, you could craft a custom IAM policy for the support role with more granular permissions. However, this is only recommended for advanced use cases with specific requirements. The managed policy covers most needs.

Some organizations may grant a wider admin/owner role to designated support personnel which makes a dedicated role unnecessary. But this violates least privilege and provides access far beyond engaging AWS support.

## Explore Further
- [AWS Support Access Policies](https://docs.aws.amazon.com/awssupport/latest/user/access-policies.html) 
- [AWS IAM Roles](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html)
- [CIS Control 17.1 - Designate Personnel to Manage Incident Handling](https://cas.docs.cisecurity.org/en/latest/source/Controls17/)