# 5.5 Ensure routing tables for VPC peering are "least access"

## Summary
When you set up VPC peering to connect two VPCs, you need to update the routing tables to actually allow traffic to flow between them. But here's the thing - you can (and should) be super specific about exactly what traffic you allow. We're talking locking it down to just the bare minimum required subnets or even single hosts if possible. Keeping it on a tight leash is the way to go!

## Where did this come from?
This sage advice comes straight from the CIS Amazon Web Services Foundations Benchmark v3.0.0 - 01-31-2024. You can grab yourself a copy of the full benchmark over at https://downloads.cisecurity.org/#/. The benchmark goes into more detail on the recommendation and links to some handy AWS docs too, like:
- https://docs.aws.amazon.com/AmazonVPC/latest/PeeringGuide/peering-configurations-partial-access.html 
- https://docs.aws.amazon.com/cli/latest/reference/ec2/create-vpc-peering-connection.html

## Who should care? 
This one is most relevant for:
- Network admins setting up and maintaining VPC peering connections
- Security engineers wanting to minimize the blast radius if a peered VPC is compromised
- Compliance officers making sure peering adheres to the principle of least privilege

## What is the risk?
The main risk is that if an attacker compromises a peered VPC, overly broad routing tables could allow them to pivot and access resources in the other VPC that they shouldn't be able to reach. Take two VPCs named Acme and Bravo. If Bravo allows a whole /16 CIDR range of Acme subnets to access it, but in reality only needs one /24 subnet for the app, a compromise of an unrelated part Acme could lead to a compromise of Bravo. 

More specific routes act as network segmentation to contain breaches. They're not a silver bullet of course, but they meaningfully reduce risk by limiting what attackers can reach.

## What's the care factor?
Admins should care a moderate to high amount about this. The effort required to limit the routes is fairly low in most cases - just a bit of upfront planning and config. But the reduction in risk can be significant, especially in larger environments with many peered VPCs. I'd recommend this as a quick win to improve security posture.

The only reason not to care as much is if you have a very simple, small setup where VPCs need full access to each other anyway. But it's still worth considering whether that itself is really needed.

## When is it relevant?
Tweaking routing tables to be least access makes sense when:
- You have multiple VPCs peered together 
- The peered VPCs only need access to a subset of each other's resources
- You want to minimize the impact of a breach in one VPC on the others

It's less relevant if:
- You only have standalone, unpeered VPCs (no need for customroutes)
- Your peered VPCs truly need unrestricted access to each other
- You're using a different method of private connectivity like TransitGateway, PrivateLink etc.

## What are the trade offs?
There are a few potential downsides to be aware of:
- Granular routing requires more planning and a deeper understanding of your app's connectivity needs 
- More complexity to manage a larger number of specific routes
- Could break things if you lock it down too much and block required traffic
- Makes peering more brittle - new subnets aren't automatically accessible

But for most, the security benefits should outweigh these as long as you test and monitor.

## How to make it happen?
For new peering connections:
1. Before you create the peering connection, map out exactly which subnets/CIDR ranges need to communicate 
2. Create the peering connection itself via the console, API, CLI etc.
3. Add specific routes to the relevant routing tables allowing only the required subnets, not the full VPC range

For existing peering connections:
1. Review your routing tables to check if they are allowing broader access than needed 
2. For any non-compliant route tables:
    - Make note of the current routes 
    - Delete overly broad routes (e.g full VPC CIDR)
    - Add in more specific routes for only the required subnets
3. Thoroughly test connectivity to make sure nothing is unexpectedly broken

You can review and update the routing tables in the VPC console area. Or use the CLI:

List route tables:
```
aws ec2 describe-route-tables --filter "Name=vpc-id,Values=<vpc_id>"
```

Delete overly broad route:
```
aws ec2 delete-route --route-table-id <route_table_id> --destination-cidr-block <non_compliant_destination_CIDR>
```

Add more specific route:
```  
aws ec2 create-route --route-table-id <route_table_id> --destination-cidr-block <compliant_destination_CIDR> --vpc-peering-connection-id <peering_connection_id>
```

## What are some gotchas?
A few things to watch out for:
- You need permissions to modify routing tables (like `ec2:CreateRoute`, `ec2:DeleteRoute` etc.)
- VPC peering routes can coexist with other types (ephemeral gateways, VPN etc). Make sure no other route can override your specific ones.
- If using longest prefix match mode, most specific route wins. Order doesn't matter.
- If you change a VPC's CIDR, make sure to update the peering routes.

## What are the alternatives?
Some other ways you can reduce the blast radius of compromised peered VPCs:
- Use different methods of private connectivity like PrivateLink, TransitGateway which allow even more granular segmentation
- Implement strict security groups and NACLs within VPCs to create many isolated tiers 
- For ultra-sensitive workloads, just don't peer the VPC at all. Use public APIs if connectivity is required.

## Explore further
- CIS AWS Benchmark 3.0.0 [Recommendation 5.5](https://www.cisecurity.org/benchmark/amazon_web_services/)
- AWS [VPC Peering docs](https://docs.aws.amazon.com/vpc/latest/peering/what-is-vpc-peering.html)

This recommendation relates to CIS control v8 3.3 and v7 14.6 which require configuring least privilege access control lists.