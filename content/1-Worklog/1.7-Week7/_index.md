---
title: "Worklog Week 7"
date: 2026-06-01
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---

### Week 7 Goals:

- Master **AWS KMS**: customer-managed key, key policy, grant, alias and rotation.
- Deploy detection & protection services: **GuardDuty**, **Security Hub**, **WAF** and **Shield Standard**.
- Get familiar with monitoring via **CloudTrail**, **CloudWatch Logs/Metrics/Alarms** and **EventBridge**.

> 5 sessions in a row — the most "security-feeling" week so far because almost every service is directly tied to protection and monitoring.

### Tasks for this week:

| Day | Task | Start date | End date | Reference |
| --- | --- | --- | --- | --- |
| 2 | - Reviewed the AWS security model: **shared responsibility model** <br> - **AWS KMS** overview: symmetric key, asymmetric key, alias, key policy <br> - Created a **customer-managed CMK** with alias `fcaj-key`, enabled **annual key rotation** <br> - Used the CMK to encrypt an EBS volume on a new EC2 and verified it in the console | 01/06/2026 | 01/06/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 3 | - Used the CMK to encrypt an S3 bucket and an RDS snapshot <br> - Practiced granting key usage to an IAM role via CloudFormation <br> - Revoked the grant and confirmed EC2 could no longer decrypt the EBS volume | 02/06/2026 | 02/06/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 4 | - Enabled **AWS CloudTrail** in a region as a multi-region trail, logs into a dedicated S3 bucket <br> - Enabled **CloudTrail Insights** and **CloudTrail Lake** <br> - Created a **CloudWatch Logs group** for CloudTrail and a metric filter counting `ConsoleLogin` events <br> - Created a **CloudWatch Alarm** sending an SNS notification when a `Root` login occurs | 03/06/2026 | 03/06/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 5 | - Enabled **GuardDuty** in `ap-southeast-1` with **S3 Protection** and **RDS Protection** <br> - Simulated an attack: ran SSH brute-force from a suspicious IP and waited for the `SSH brute force` finding <br> - Enabled **Security Hub** in the region and linked GuardDuty, IAM Access Analyzer, Inspector into a single dashboard <br> - Reviewed all findings and ranked them by severity | 04/06/2026 | 04/06/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 6 | - Deployed an ALB in front of the web-tier EC2 <br> - Attached **AWS WAF** to the ALB with a rule blocking requests containing `User-Agent: BadBot` <br> - Added a **rate-based rule** blocking any IP sending more than 2000 requests / 5 minutes <br> - Enabled **Shield Standard** (free) for the ALB <br> - Cleanup: removed test rules, disabled GuardDuty to avoid charges | 05/06/2026 | 05/06/2026 | <https://cloudjourney.awsstudygroup.com/> |

### Week 7 Outcomes:

- Solid understanding of the **shared responsibility model** and clear separation between AWS and customer responsibilities.
- Created and managed a **KMS customer-managed key** with key policy, alias and rotation.
- Used KMS to encrypt EBS, S3 and RDS snapshot.
- Deployed CloudTrail multi-region, CloudWatch Logs/Metrics/Alarms with SNS.
- Enabled **GuardDuty**, simulated SSH brute-force, and read the generated finding.
- Enabled **Security Hub** and consolidated findings from multiple security services.
- Attached **WAF** to an ALB with bot-blocking and rate-based rules.

### Difficulties and lessons learned:

- WAF cannot be attached directly to an EC2 — there must be an **Application Load Balancer** or **CloudFront** in front. A good lesson in how AWS designs defense in depth.
- CloudTrail Insights needs a baseline period to detect anomalies, so you have to wait a few days before meaningful alerts appear.
- GuardDuty takes around 5-15 minutes to flag SSH brute-force, so "real-time" testing requires patience.