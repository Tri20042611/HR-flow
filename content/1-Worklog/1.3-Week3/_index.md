---
title: "Worklog Week 3"
date: 2026-05-04
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---

### Week 3 Goals:

- Master **Amazon S3**: buckets, objects, storage classes, versioning, lifecycle and replication.
- Practice S3 from a security perspective: Block Public Access, pre-signed URL, SSE-KMS and Bucket Policy.
- Compare storage services to choose the right one for each use case.

> I had 4 hands-on sessions this week, all focused on S3 since it shows up the most in security scenarios.

### Tasks for this week:

| Day | Task | Start date | End date | Reference |
| --- | --- | --- | --- | --- |
| 2 | - S3 overview: bucket, object, key, prefix, global bucket naming rules <br> - Storage classes: Standard, Standard-IA, One Zone-IA, Glacier Instant Retrieval, Glacier Deep Archive <br> - Hands-on: created the first bucket, uploaded via console & CLI | 04/05/2026 | 04/05/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 3 | - Enabled **Versioning** on the bucket <br> - Configured **Lifecycle rule** to transition to Glacier after 30 days and expire after 365 days <br> - Configured **Cross-Region Replication (CRR)** to a bucket in another region <br> - Set up **Event notification** to an SNS topic on object creation | 05/05/2026 | 05/05/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 5 | - Enabled **Block Public Access** at both account and bucket levels <br> - Wrote a **Bucket Policy** granting a specific IAM role read access without pre-signed URL <br> - Generated a **pre-signed URL** with 15-minute expiry for internal file sharing <br> - Enabled **Server Access Logging** and **CloudTrail Data Events** for S3 <br> - Tried to access with an IAM user without `s3:GetObject` to confirm the policy actually denies | 07/05/2026 | 07/05/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 6 | - Enabled **Default encryption with SSE-KMS** using a customer-managed key <br> - Tried to access with an IAM user without `kms:Decrypt` to confirm policy behavior <br> - Created **EFS**, mounted from two EC2s in different AZs, wrote from one and read from the other <br> - Compared **EBS / EFS / FSx / S3 / Glacier** to map each to the right use case | 08/05/2026 | 08/05/2026 | <https://cloudjourney.awsstudygroup.com/> |

### Week 3 Outcomes:

- Created and managed S3 buckets confidently; understood global bucket naming and region selection.
- Configured Versioning, Lifecycle rule and Cross-Region Replication successfully.
- Practiced key S3 security controls:
  - Block Public Access as the final safety net against public exposure.
  - Pre-signed URL for temporary file sharing without making objects public.
  - SSE-KMS combined with IAM policies to control decryption rights.
- Compared and chose between **EBS / EFS / FSx / S3 / Glacier** for real use cases.
- Explained common S3 security pitfalls (misconfig, public bucket, missing logs) and how to prevent them.

### Difficulties and lessons learned:

- Cross-Region Replication requires both buckets to enable Versioning; I forgot that and replication failed.
- When writing Bucket Policy, always check the `Condition` block with `aws:SourceIp` or `aws:PrincipalArn` to avoid granting more than intended.
- Pre-signed URLs default to a 1-hour expiry; tune it depending on the use case.