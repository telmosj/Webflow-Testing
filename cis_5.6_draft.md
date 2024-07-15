# 5.6 Ensure that EC2 Metadata Service only allows IMDSv2 (Automated)

## Summary
The EC2 Instance Metadata Service (IMDS) provides a way for EC2 instances to retrieve metadata about themselves, such as their instance ID, private IP address, and IAM role. However, the original version of the service, IMDSv1, is vulnerable to server-side request forgery (SSRF) attacks. To mitigate this risk, Amazon recommends configuring EC2 instances to use the more secure IMDSv2, which requires session-based authentication for each metadata request.

## Where did this come from?
This recommendation comes from the CIS Amazon Web Services Foundations Benchmark v3.0.0 - 01-31-2024, which can be downloaded from the [CIS website](https://downloads.cisecurity.org/#/). The benchmark provides prescriptive guidance for configuring security options for a subset of Amazon Web Services with an emphasis on foundational, testable, and architecture agnostic settings.

For more background on the risks of IMDSv1 and the benefits of IMDSv2, check out this [AWS Security Blog post](https://aws.amazon.com/blogs/security/defense-in-depth-open-firewalls-reverse-proxies-ssrf-vulnerabilities-ec2-instance-metadata-service/). The [EC2 user guide](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/configuring-instance-metadata-service.html) also provides an overview of the instance metadata service and instructions for configuring the different versions.

## Who should care? 
This recommendation is relevant for:
- DevOps engineers responsible for provisioning and configuring EC2 instances
- Security engineers looking to harden EC2 instances against SSRF attacks
- Compliance officers ensuring adherence to CIS benchmarks and other security standards

## What is the risk?
The main risk of allowing IMDSv1 is that it enables server-side request forgery (SSRF) attacks. In an SSRF attack, a malicious actor tricks a vulnerable application running on an EC2 instance into making requests to the instance metadata service. These requests can retrieve sensitive data like IAM role credentials, which the attacker can then use to gain unauthorized access to other AWS services and resources.

While IMDSv2 does not completely eliminate the possibility of SSRF, it makes attacks much harder by requiring a session token to be included with each metadata request. This means an attacker would need to compromise the application to the point where they can generate valid session tokens, rather than just tricking it into making an HTTP request to the metadata service URL.

## What's the care factor?
For most organizations, switching to IMDSv2 should be considered a high priority. SSRF attacks can lead to serious breaches, especially if the compromised EC2 instance has an IAM role with broad permissions. The effort required to transition to IMDSv2 is relatively low, especially for newer EC2 instances, and the improved security posture is well worth it.

That said, if you have a fleet of older EC2 instances running legacy applications that would be difficult to update, it may make sense to prioritize other security improvements first. Just be aware of the heightened risk and take other precautions like tightly scoping IAM roles and monitoring for suspicious activity.

## When is it relevant?
Configuring IMDSv2 is recommended for all EC2 instances. Even if you don't think your instance is exposed to the internet or running untrusted code, it's still a worthwhile security best practice.

However, if you are running EC2 instances in a private subnet with no internet access, the risk of SSRF may be lower since it would require an attacker to already have a foothold in your VPC. In this case, you could potentially deprioritize the switch to IMDSv2 in favor of other tasks.

## What are the trade offs?
The main cost of enabling IMDSv2 is the time and effort required to update your application code and/or EC2 launch configurations to use the new session-based authentication. This is generally pretty straightforward, but it may require some testing to ensure compatibility.

There is a slight performance overhead for IMDSv2 since it requires an additional request to generate a session token. However, this is negligible for most use cases.

Finally, keep in mind that instances launched before a certain date (October 2019) do not support IMDSv2. So if you have a lot of older instances, you may need to plan for a longer transition period or prioritize updating them to a newer version.

## How to make it happen?
To transition an EC2 instance to IMDSv2, you need to update two things:

1. Modify the instance metadata options to require IMDSv2. You can do this from the EC2 console under Actions > Instance Settings > Modify instance metadata options. Set the " IMDSv2" option to "Required". Alternatively, use the AWS CLI command:

   ```
   aws ec2 modify-instance-metadata-options --instance-id <instance-id> --http-tokens required --region <region>  
   ```

2. Update your application code to retrieve a session token before making metadata requests. The process looks like:

   a. Generate a session token with a PUT request to `http://169.254.169.254/latest/api/token`. You must supply a `X-aws-ec2-metadata-token-ttl-seconds` header specifying the TTL of the token in seconds (up to 21600 seconds).

   b. Include the returned session token in the `X-aws-ec2-metadata-token` header for subsequent metadata GET requests.

   Most AWS SDKs and tools like the EC2 instance CLI have been updated to support IMDSv2 session tokens transparently. Check the documentation for your particular software.

## What are some gotchas?
The main thing to watch out for is that IMDSv2 is not supported for EC2 instances launched before the feature was released around October 2019. If you try to enable IMDSv2 on an older instance, you may find that metadata requests start failing. The only fix is to launch a new instance with IMDSv2 supported from the get-go.

When making the IMDSv2 PUT request to retrieve a session token, make sure to handle timeouts and errors gracefully. Per the [EC2 user guide](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/configuring-instance-metadata-service.html), the PUT request will fail if not completed within 1 second. Retry and backoff as needed.

Also keep in mind that to make the initial IMDSv2 configuration change via the ModifyInstanceMetadataOptions API or EC2 console, you need the `ec2:ModifyInstanceMetadataOptions` permission. Make sure the IAM user or role you are using to make the change has the necessary permissions. 

## What are the alternatives?
Realistically, there is no good alternative to enabling IMDSv2 for EC2 instances. It's the most effective way to mitigate the risk of SSRF attacks against the metadata service.

If for some reason you cannot update an instance to use IMDSv2, you should take other steps to reduce the blast radius of a potential compromise:
- Use granular IAM roles and avoid overly broad permissions
- Implement network controls to limit outbound internet access from the instance 
- Monitor CloudTrail and VPC Flow Logs for suspicious activity
- Regularly rotate and limit the scope of credentials stored on the instance

But these are all band-aids compared to fixing the root issue and requiring IMDSv2.

## Explore Further
- [CIS Amazon Web Services Foundations Benchmark v3.0.0 - 01-31-2024](https://downloads.cisecurity.org/#/)

- [EC2 User Guide: Configure the instance metadata service](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/configuring-instance-metadata-service.html)

- [AWS Security Blog: Defense in depth: Open firewalls, reverse proxies, SSRF, metadata services](https://aws.amazon.com/blogs/security/defense-in-depth-open-firewalls-reverse-proxies-ssrf-vulnerabilities-ec2-instance-metadata-service/)

- [EC2 Instance Metadata and User Data](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-metadata.html)

This recommendation overlaps with a few related CIS controls:
- CIS Control 3 - Data Protection: Metadata services can potentially expose sensitive data if not properly secured 
- CIS Control 4 -Secure Configuration of Enterprise Assets and Software