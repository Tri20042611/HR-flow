---
title: "Worklog Week 4"
date: 2026-05-11
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

### Week 4 Goals:

- Deep dive into **Amazon VPC**: CIDR, subnet (public/private), route table, Internet Gateway, NAT Gateway.
- Build a multi-tier VPC (public web tier, private app + db tier) and trace packets from outside in.
- Learn Security Group vs NACL, VPC Peering, Route 53 and out-of-VPC connectivity (VPC Endpoint, PrivateLink).

> The heaviest week so far with 5 sessions back-to-back because VPC is the foundation of everything. Spent a lot of time debugging route tables and security groups.

### Tasks for this week:

| Day | Task | Start date | End date | Reference |
| --- | --- | --- | --- | --- |
| 2 | - VPC overview: CIDR block, what a VPC is and how it differs from a subnet <br> - Difference between public and private subnet <br> - Used the `VPC and more` wizard to launch the first VPC and reviewed the auto-created resources | 11/05/2026 | 11/05/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 3 | - **Internet Gateway (IGW)** and **Route Table**: how packets flow from Internet to a public subnet via IGW <br> - Created 2 public subnets in 2 different AZs <br> - Configured route table pointing `0.0.0.0/0` to IGW for the public subnets <br> - Launched EC2 in the public subnet, verified Internet access via `curl ifconfig.me` | 12/05/2026 | 12/05/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 4 | - **NAT Gateway** vs **NAT Instance**: why private subnet needs NAT to reach the Internet <br> - Created private subnets in 2 AZs <br> - Deployed NAT Gateway in the public subnet, routed `0.0.0.0/0` from the private subnets to NAT <br> - Hands-on: an EC2 in the private subnet can `apt update` through NAT but cannot be SSH-ed from outside | 13/05/2026 | 13/05/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 5 | - Difference between **Security Group (stateful)** and **Network ACL (stateless)** <br> - Wrote a custom NACL: deny port 22 but allow 80/443 <br> - Deployed Apache + MariaDB on 3 EC2s in a web - app - db tier model <br> - Tested: web tier reaches the Internet, app tier only accepts traffic from web, db tier only accepts traffic from app | 14/05/2026 | 14/05/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 6 | - **VPC Peering** and **VPC Endpoint** overview <br> - Created an Interface VPC Endpoint for S3, confirmed the EC2 in the private subnet reaches S3 without going through NAT Gateway <br> - AWS DNS basics: **Route 53** hosted zone, A, CNAME, Alias records <br> - Cleaned up the entire VPC to avoid charges | 15/05/2026 | 15/05/2026 | <https://cloudjourney.awsstudygroup.com/> |

### Week 4 Outcomes:

- Solid understanding of multi-tier VPC and able to trace packets from the user through web → app → db.
- Designed VPC and subnet CIDR blocks from scratch, leaving room for future growth.
- Configured **Internet Gateway**, **NAT Gateway** and route tables for public/private subnets.
- Clearly differentiated **Security Group (stateful)** vs **Network ACL (stateless)** and know when to use which.
- Deployed an **Interface VPC Endpoint** for S3 to reduce NAT cost and keep traffic private.
- Got the basics of **Route 53** hosted zones and the most common record types (A, AAAA, CNAME, Alias).

### Difficulties and lessons learned:

- Drawn the wrong route table for the private subnet pointing straight to IGW instead of NAT, which exposed the EC2's public IP. Strong reminder of the **least privilege** principle in networking.
- NAT Gateway is billed per hour, so it must be deleted after practice to avoid surprise charges.
- NACL is stateless, so both **inbound and outbound** rules must allow ephemeral ports — easy to miss versus Security Group.