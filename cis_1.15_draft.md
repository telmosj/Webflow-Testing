# Ensure IAM Users Receive Permissions Only Through Groups (Automated)

## Summary
Managing IAM permissions for users in AWS can get messy fast if you're not careful. The best way to keep things tidy is to assign permissions to IAM groups, then add users to those groups. Resist the temptation to attach policies directly to users or edit their inline policies - that way lies chaos!

## Where did this come from?
This recommendation comes straight from the "CIS Amazon Web Services Foundations Benchmark v3.0.0 - 01-31-2024". You can download a copy of the full benchmark document from the [CIS website downloads page](https://downloads.cisecurity.org/#/).

For more background, check out the AWS IAM documentation on [IAM Best Practices](http://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html) and [Managed Policies vs Inline Policies](http://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_managed-vs-inline.html).

## Who should care?
This one is relevant for:
* IAM Administrators responsible for managing AWS permissions
* Compliance officers ensuring adherence to security best practices
* Developers and DevOps engineers setting up IAM for their projects

## What is the risk?
The main risks of assigning policies directly to users are:
* Lack of consistency - It's hard to ensure users have the right level of access when each one has a custom policy attached
* Excessive permissions - One-off user policies tend to err on the side of too much access "just in case"
* Difficulty managing permissions over time - Figuring out who can access what becomes a major headache

Using IAM groups helps mitigate these issues by providing a central mechanism to manage permissions based on job functions.

## What's the care factor? 
For organizations with more than a handful of IAM users, I'd rate this as a "high" priority recommendation. The effort to set up groups is fairly minimal, and the payoff in simplified permissions management is significant.

However, very small or early-stage setups may have more pressing priorities. As always, weigh the benefits against your organization's specific risk tolerance and resources.

## When is it relevant?
IAM groups make sense for almost any AWS setup with multiple users. They're especially helpful for growing organizations where you need a scalable way to manage permissions over time.

The only situation where you might forego groups is a very simple setup, e.g. a personal account or small project with 2-3 users max. Even then, getting in the habit of using groups can pay off down the road.

## What are the trade-offs?
Implementing IAM groups does take some upfront planning and effort. You'll need to:
* Analyze your permission requirements and design appropriate groups 
* Create the groups and attach policies
* Audit existing users and move them into the right groups
* Update processes for onboarding new users

There may also be some user friction as people adjust to the new permissions model. Be prepared for questions and have a plan to handle requests/issues.

## How to make it happen?

Here's how to implement the recommendation in your AWS account:

1. Design your permission groups:
    - List out the different job functions/roles in your org
    - Map out the permissions needed for each role
    - Look for commonalities to define broad groups (e.g. Admins, Developers, Analysts)
    - Fine-tune groups as needed with more specific policies

2. Create the IAM groups:
    - Open the IAM console and navigate to Groups
    - Click "Create New Group"
    - Name the group based on role/function (e.g. NetworkAdmins)
    - Attach the appropriate managed policies to the group
    - Click "Create Group"
    - Repeat for each group in your design

3. Add users to groups:
    - Go to the IAM user list
    - Click on each user, go to the "Groups" tab
    - Click "Add user to groups"  
    - Select the appropriate group(s) and click "Add to Groups"

4. Remove existing inline & direct user policies:
    - For each user, click on "Permissions"
    - Expand the "Permissions policies" section
    - If any policies are listed, click the "X" to remove them
    - Confirm the removal when prompted

5. Update new user provisioning process:
    - Modify IAM user creation templates/scripts to only include group memberships
    - Ensure IAM admins understand to provision access via groups only going forward
    - Consider disabling the policy attachment options via IAM permissions boundaries

## What are some gotchas?
Some things to watch out for when implementing IAM groups:

- Make sure you have a thorough understanding of your permission requirements before designing the groups. Missing requirements can result in overly broad policies.

- When moving existing users to groups, be careful not to inadvertently remove access they may need. Compare group policies to their existing policies.

- You'll need IAM permissions to manage groups and group membership. Key actions include:
    - iam:CreateGroup, iam:ListGroups, iam:ListGroupsForUser
    - iam:AddUserToGroup, iam:RemoveUserFromGroup
    - iam:AttachGroupPolicy, iam:DetachGroupPolicy
    - iam:PutGroupPolicy, iam:DeleteGroupPolicy
    - See the [AddUserToGroup API docs](https://docs.aws.amazon.com/IAM/latest/APIReference/API_AddUserToGroup.html) for details.

- Group names can be changed, but the ARN will change, which may break things (e.g. resource policies). Avoid renaming groups if possible.

- If using AWS Organizations, be aware that group membership can't be used as a condition for Service Control Policies. User-level SCPs override group permissions.

## What are the alternatives?
If you have a large number of users or complex permission requirements, you might consider some alternatives/complements to groups:

- IAM Roles - For granting temporary access to AWS resources, IAM roles can be assumed by users without having to be explicitly added/removed. Useful for cross-account access.

- Attribute-Based Access Control (ABAC) - Rather than defining permissions based on job function, ABAC grants access based on attributes like department, project, environment, etc. Helpful for very granular, dynamic permission requirements. Requires more setup.

- AWS SSO - If you're using an external identity provider, you can define group mappings and permission sets in AWS SSO. Manages the complexity outside of IAM.

## Explore Further
- CIS Benchmark Recommendation 1.16 goes hand-in-hand with this one - "Ensure IAM policies that allow full "*:*" administrative privileges are not created"

- Check out the [AWS IAM Best Practices Guide](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html) for more tips on locking down access.

- To take things even further, explore setting up an [authorization model using IAM Permissions Boundaries](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_boundaries.html).