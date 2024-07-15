# 1.19 Ensure that all the expired SSL/TLS certificates stored in AWS IAM are removed (Automated)

## Summary
Hey there! Did you know that expired SSL/TLS certificates stored in your AWS IAM can be a security risk? Yep, it's true. The best practice is to regularly check for and remove any expired certs to keep your applications and websites secure. Let's dig into the details!

## Where did this come from?
This recommendation comes straight from the "CIS Amazon Web Services Foundations Benchmark v3.0.0 - 01-31-2024". You can download the full benchmark document from the Center for Internet Security (CIS) website here: https://downloads.cisecurity.org/#/. The benchmark provides a ton of great AWS security best practices and recommendations. The AWS documentation on managing server certificates in IAM is also a handy reference: https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_server-certs.html

## Who should care? 
This is most relevant for:
- AWS administrators responsible for managing IAM and SSL/TLS certificates
- Application owners using SSL/TLS certificates stored in IAM for their AWS-hosted apps/websites 
- Security teams overseeing the security posture of AWS accounts and resources

## What is the risk?
The main risk is accidentally deploying an expired certificate to a resource like an Elastic Load Balancer, which can damage the credibility and availability of the application or website. Imagine hitting your app and getting one of those scary "Warning: Potential Security Risk Ahead" messages because of an expired cert! Not a great look. While the likelihood may be low if you have good cert management processes, the impact to reputation could be significant.

## What's the care factor?
For public-facing production apps and websites, the care factor for this should be high. It's a pretty low effort task that can prevent some really bad situations.  Maybe less critical for internal-only dev/test systems, but in general, expired certs lingering around is not good hygiene. An ounce of prevention is worth a pound of cure as they say!

## When is it relevant?
This recommendation is relevant anytime you are storing and using SSL/TLS certificates in IAM. Some common use cases:
- Elastic Load Balancers (ELB) 
- CloudFront distributions
- API Gateway endpoints
It's less relevant if you are using AWS Certificate Manager (ACM) to provision and manage certs, which is often a better option when supported.

## What are the trade offs?
The main downside is needing to spend time checking and cleaning up IAM certs. It shouldn't be too onerous though in most cases. The other thing to be aware of is if you delete an expired cert that is still configured on a load balancer or some other resource, it could cause an interruption until you deploy a new valid cert. So be thoughtful when removing!

## How to make it happen?
Here's the step-by-step:

1. List all server certificates in IAM using the AWS CLI:
```
aws iam list-server-certificates
```
2. Check the `Expiration` date for each cert in the output. Anything already expired (or maybe expiring very soon) is a candidate for deletion.

3. If you find any expired certs you want to remove, delete them with:
```
aws iam delete-server-certificate --server-certificate-name <certificate-name>
```
4. Ensure any resources (ELBs, CloudFront, etc) that were using the deleted expired cert are updated to use a new valid cert.

5. Rinse and repeat on a regular cadence!

## What are some gotchas?

- To work with certs in IAM via the CLI, you'll need IAM permissions like `iam:ListServerCertificates` and `iam:DeleteServerCertificate` 
- Again, be careful not to delete certs that are actively being used without replacing them or you could cause downtime
- If the `list-server-certificates` command returns an empty `ServerCertificateMetadataList`, it means there are NO certs in IAM, not just no expired ones

## What are the alternatives?
Wherever possible, use AWS Certificate Manager (ACM) instead of uploading certs into IAM. ACM makes provisioning, deploying and rotating certs much easier and more cost-effective. ACM integrates nicely with many AWS services like ELB, CloudFront, API Gateway and more.  See: https://docs.aws.amazon.com/acm/latest/userguide/acm-services.html

## Explore further

- AWS ACM FAQs: https://aws.amazon.com/certificate-manager/faqs/ 
- Deep dive on SSL certs in AWS: https://aws.amazon.com/blogs/security/understanding-ssl-tls-certificates/
- Related CIS Benchmark recommendations:
  - 1.20 – Ensure that IAM Access analyzer is enabled (Automated)
  - 2.10 – Ensure IAM password policy prevents password reuse (Automated)

I hope this helps explain both the "what" and the "why" behind this CIS recommendation in a more casual, practical way. Let me know if you have any other questions!