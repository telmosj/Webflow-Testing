# 3.6 Ensure rotation for customer-created symmetric CMKs is enabled (Automated)

## Summary
Hey there! Let's talk about a nifty security recommendation from the folks at the Center for Internet Security (CIS). They suggest enabling automatic rotation for customer-created symmetric Customer Master Keys (CMKs) in AWS Key Management Service (KMS). This basically means regularly swapping out the secret key material used for encryption, which can help keep your data safe even if a key gets compromised. Pretty cool, right?

## Where did this come from?
This recommendation comes straight from the "CIS Amazon Web Services Foundations Benchmark v3.0.0 - 01-31-2024". You can download the full benchmark document chock-full of other useful security tips from the CIS website: [https://downloads.cisecurity.org/#/](https://downloads.cisecurity.org/#/). For more juicy details on KMS key rotation, check out the AWS docs: [AWS Key Management Service Developer Guide - Rotating customer master keys](https://docs.aws.amazon.com/kms/latest/developerguide/rotate-keys.html).

## Who should care?
This is a biggie for:
- Security engineers responsible for locking down AWS environments 
- Compliance officers ensuring adherence to security best practices
- Developers using AWS KMS for application-level encryption 

## What is the risk?
The main risk here is that if an attacker gets their hands on one of your encryption keys, they could decrypt sensitive data that was encrypted with that key. Yikes! Regularly rotating keys limits the blast radius since new data will be encrypted with a fresh key. While rotating keys won't stop a breach, it can certainly help contain the damage. The likelihood of a key compromise depends on your security practices, but the consequences could be quite severe - imagine all your secret customer data exposed! 

## What's the care factor?
Key rotation is one of those security fundamentals that everyone should implement. It's not a silver bullet, but it's a relatively low-effort way to reduce risk. I'd rate it as a "high" priority, especially for organizations dealing with sensitive data like personally identifiable information (PII), financial records, or health info. Seriously, don't sleep on this one!

## When is it relevant? 
You'll want to enable automatic key rotation for any symmetric CMKs you've created in KMS. It's especially important for keys used to encrypt sensitive data at rest. However, there are a couple situations where key rotation may not apply:
- Asymmetric CMKs used for public key encryption or signing (KMS doesn't support rotating these)
- CMKs that aren't actively being used to encrypt new data (rotating them won't really do anything) 
- Super short-lived data where you rotate the entire dataset frequently (e.g. ephemeral caches)

## What are the trade-offs?
The main downside to key rotation is a bit of added complexity and cost:
- You'll be paying for KMS to store multiple versions of each key ($1/month per key)  
- Some extra automation logic may be needed to handle key rotation events
- A little more to think about when encrypting/decrypting data
But honestly, for most use cases the benefits far outweigh these minor drawbacks. The peace of mind alone is worth it!

## How to make it happen?
Enabling automatic key rotation in KMS is actually pretty simple:

1. Open the KMS console and navigate to "Customer managed keys" 
2. Find the symmetric CMK you want to enable rotation for
3. Select the "Key rotation" tab for that key
4. Check the box that says "Automatically rotate this KMS key every year"
5. Click the "Save" button

Boom! You're done. KMS will now rotate that key annually without any further action on your part. All new data encrypted with the key will use the latest backing key, while KMS transparently decrypts old data with previous key versions. Slick!

If you prefer the command line, you can also enable rotation with a single AWS CLI command:

```
aws kms enable-key-rotation --key-id <your-key-id>
```

## What are some gotchas?
A few things to watch out for when enabling key rotation:
- Make sure you actually created the CMK yourself. You can't rotate AWS managed keys.
- Key rotation only applies to customer-created symmetric CMKs (asymmetric ones aren't supported) 
- Enabling rotation requires the `kms:EnableKeyRotation` permission on the key policy or IAM
- Some AWS services that integrate with KMS may not be compatible with automatic key rotation (double check the docs for your specific use case)

## What are the alternatives?
If you don't want to enable automatic key rotation in KMS, you could:
- Manually rotate keys by creating a new CMK and re-encrypting your data periodically 
- Use application-level key rotation in your own code (more work, but more control)
- Accept the risk and not rotate keys (not recommended for important data!)

## Explore further
Intrigued? Here are some additional resources to go deeper on KMS key rotation:
- [AWS KMS Cryptographic Details Whitepaper](https://d0.awsstatic.com/whitepapers/KMS-Cryptographic-Details.pdf) 
- [CIS Controls v8 3.11 - Encrypt Sensitive Data at Rest](https://www.cisecurity.org/controls/cis-controls-navigator/v8) 
- [NIST SP 800-57 Recommendation for Key Management](https://csrc.nist.gov/publications/detail/sp/800-57-part-1/rev-5/final)

And that's the scoop on CIS 3.6! By enabling automatic KMS key rotation, you'll be well on your way to better protecting your most valuable AWS assets. Happy encrypting!