# 2.3.2 Ensure Auto Minor Version Upgrade feature is Enabled for RDS Instances (Automated)

## Summary
In a casual tone, it's important to make sure your Amazon RDS database instances have the Auto Minor Version Upgrade feature turned on. This allows RDS to automatically apply minor version upgrades with new features, bug fixes, and security patches during the scheduled maintenance window. Keeping your RDS instances up-to-date helps keep them secure and running smoothly.

## Where did this come from? 
This recommendation comes from the CIS Amazon Web Services Foundations Benchmark v3.0.0 - 01-31-2024, which you can download from https://downloads.cisecurity.org/#/. The CIS Benchmarks are best practices for securely configuring IT systems, and the AWS Foundations Benchmark provides recommendations specific to AWS services. You can find more information in the [AWS RDS documentation](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Welcome.html).

## Who should care?
This recommendation is most relevant for:
- Database administrators responsible for managing RDS instances
- DevOps engineers deploying applications backed by RDS databases 
- Security teams ensuring cloud resources adhere to security best practices
- Compliance officers verifying alignment with industry standards

## What is the risk?
By not enabling auto minor version upgrades, your RDS instances may be running database engines with known security vulnerabilities that have been patched in newer versions. This leaves you exposed to potential exploits. Falling behind on new features and optimizations can also degrade database performance over time.

While unlikely, a severe unpatched vulnerability could allow an attacker to gain unauthorized access to your database. Data breaches can result in sensitive information disclosure, financial losses, and reputational damage.

## What's the care factor? 
For production RDS instances holding sensitive data, enabling auto minor version upgrades should be considered mandatory from a security perspective. The effort required is minimal compared to the risk of running vulnerable database versions.

For non-production or less critical instances, auto upgrades are still strongly recommended as general best practice to maintain a consistent and secure environment. The only reason to potentially delay upgrades is if you have legacy applications that are incompatible with newer database versions.

## When is it relevant?
Auto minor version upgrades should be enabled for:
- All production RDS instances
- Non-production instances used for staging or testing 
- Database engines where you want to stay current with features/fixes

It may not be necessary for:
- Isolated dev environments that don't hold real data
- Legacy applications that can only run on a specific database version

## What are the trade offs?
Enabling auto minor version upgrades does incur some potential trade-offs:
- Upgrades require databases to restart, causing 1-2 mins of downtime. Schedule the maintenance window during low activity periods to minimize impact.
- Rarely, newer database versions could introduce incompatibilities or change behavior. Thoroughly test upgrades in non-prod first.
- If you delay applying upgrades, you have more control over the timing and cadence of changes vs accepting them automatically each month.

## How to make it happen?
To enable auto minor version upgrades for an RDS instance:

1. Open the [AWS RDS console](https://console.aws.amazon.com/rds/)
2. Click "Databases" in the left nav and select the target DB instance 
3. Click the "Modify" button in the top right
4. Under "Maintenance", select "Yes" for "Auto minor version upgrade"
5. Choose whether to apply the change immediately or during the next maintenance window
6. Click "Continue", review the change summary, and click "Modify DB Instance"

Alternatively, use this AWS CLI command:

```
aws rds modify-db-instance --db-instance-identifier <instance-id> --auto-minor-version-upgrade --apply-immediately
```

## What are some gotchas?
A few things to watch out for when enabling auto minor version upgrades:
- You need the `rds:ModifyDBInstance` permission to change this setting
- If you previously had auto upgrades disabled, there may be multiple versions to catch up on, so plan that into your maintenance cycle
- Some older DB instances (2017 and earlier) may not support auto upgrades. Follow the manual upgrade process instead.
- If you use read replicas, upgrade the source instance first, then upgrade read replicas after they catch up to avoid replication issues. 

## What are the alternatives?
Instead of enabling auto upgrades, you can manually initiate minor version upgrades on your own schedule:

1. Follow the AWS docs to [manually upgrade the DB engine version](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_UpgradeDBInstance.Upgrading.html) 
2. Use AWS Systems Manager Maintenance Windows to [schedule automated upgrades via runbooks](https://aws.amazon.com/blogs/database/schedule-amazon-rds-stop-and-start-using-aws-systems-manager/)

However, this is more effort than auto upgrades and risks falling behind on versions if not done regularly. In most cases auto upgrades are the way to go.

## Explore further
- Read the [RDS User Guide](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Welcome.html) to learn more about available features and best practices
- See the AWS docs on [automatically upgrading the minor engine version](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_UpgradeDBInstance.Upgrading.html#USER_UpgradeDBInstance.Upgrading.AutoMinorVersionUpgrades)
- Review additional security recommendations in the [CIS AWS Foundations Benchmark](https://www.cisecurity.org/benchmark/amazon_web_services/)
- Minor version upgrades are part of a broader configuration management strategy - see [CIS Control 7.4 for Automated Application Patch Management](https://www.cisecurity.org/controls/cis-controls-navigator)