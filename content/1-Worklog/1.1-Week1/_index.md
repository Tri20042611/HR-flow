---
title: "Worklog Week 1"
date: 2026-04-20
weight: 1
chapter: false
pre: " <b> 1.1. </b> "
---

### Week 1 Goals:

- Get to know members, mentors and the rules of the First Cloud AI Journey program.
- Understand AWS overview and main service groups (Compute, Storage, Networking, Database, Security).
- Create an AWS Free Tier account, get familiar with AWS Management Console, and install AWS CLI.

> The first week schedule was light. I only had 3 real working sessions on Monday, Wednesday and Friday. The other days were spent reading docs at home and handling personal stuff.

### Tasks done this week:

| Session | Date | Task | Reference |
| --- | --- | --- | --- |
| Session 1 | 20/04/2026 | Kick-off: <br> - Introduced myself to the team, got assigned a mentor <br> - Signed the NDA and reviewed internship regulations <br> - Got the 12-week roadmap overview | |
| Session 2 | 22/04/2026 | Hands-on with AWS: <br> - Created AWS Free Tier account, enabled MFA on root <br> - Created a dedicated IAM user instead of using root for daily work <br> - Installed AWS CLI v2, ran `aws configure` with region `ap-southeast-1` <br> - Verified with `aws sts get-caller-identity` | <https://cloudjourney.awsstudygroup.com/> |
| Session 3 | 24/04/2026 | First EC2: <br> - EC2 basics: instance type (`t2.micro`), AMI (Ubuntu 22.04), 8GB default EBS <br> - Created a `.pem` key pair, chmod 400 on Linux <br> - Configured Security Group to open port 22 only from my personal IP <br> - SSH-ed in successfully and installed Nginx <br> - Set a Budget alert at 1 USD to prevent unexpected charges | <https://cloudjourney.awsstudygroup.com/> |

### Week 1 Outcomes:

- Understood what AWS is and its core service groups:
  - Compute (EC2, Lambda)
  - Storage (S3, EBS)
  - Networking (VPC, Route 53)
  - Database (RDS, DynamoDB)
  - Security (IAM, KMS, GuardDuty)

- Created and configured AWS Free Tier account with a dedicated IAM user for daily operations.

- Installed and configured AWS CLI, verified identity with `aws sts get-caller-identity`.

- Launched the first EC2, SSH-ed in with key pair, and ran Nginx.

- Learned how to control cost with Budget alerts and clean up resources after practice.

### Difficulties and lessons learned:

- At first I was confused about root vs IAM users. After reading docs I understood: root is for account management only, every operational action should go through an IAM user for traceability.
- I initially opened port 22 to `0.0.0.0/0` on the Security Group, then realized how dangerous that was with SSH-scanning bots. Locked it down to my IP only.
- Reading AWS English docs is still slow because I have to translate along the way.