---
title: "Worklog Week 6"
date: 2026-05-25
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

### Week 6 Goals:

- Deep dive into **IAM**: user, group, role, policy (managed vs inline), Access Analyzer and IAM Access Advisor.
- Apply **MFA** to every privileged user and configure a strong password policy.
- Learn **AWS IAM Identity Center (SSO)**, integrate with Microsoft AD and assign permission sets across accounts.

> 4 sessions this week, all focused on Identity & Access Management — a dry but extremely important topic for anyone in security.

### Tasks for this week:

| Day | Task | Start date | End date | Reference |
| --- | --- | --- | --- | --- |
| 3 | - Reviewed IAM structure: principal, action, resource, condition <br> - Differentiated **AWS managed policy**, **Customer managed policy** and **Inline policy** <br> - Created groups `Developers`, `SecurityAuditors`, `ReadOnly` and wrote dedicated policies for each based on **least privilege** | 26/05/2026 | 26/05/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 4 | - Enabled **MFA** for the root user using a hardware authenticator app on the phone <br> - Enforced MFA for the `Developers` group via an IAM policy with `aws:MultiFactorAuthPresent` <br> - Configured **password policy**: minimum length 14, upper/lower/digit/special char required, 90-day expiry <br> - Tried to log in without MFA to confirm the policy actually denies | 27/05/2026 | 27/05/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 5 | - **IAM Roles** for cross-account access and AWS services (EC2, Lambda, Glue...) <br> - Created role `EC2-S3-ReadOnly` so EC2 can assume it instead of using access keys <br> - Used **IAM Access Analyzer** to detect resources shared with principals outside the organization <br> - Inspected **IAM Access Advisor** per user to see which services are actually being used and tightened policies accordingly | 28/05/2026 | 28/05/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 6 | - **AWS IAM Identity Center (SSO)** overview: why use it instead of IAM users when you have many accounts <br> - Enabled IAM Identity Center, created user `security-lead@example.com` <br> - Created **permission sets** `DeveloperReadOnly` and `SecurityAdmin` <br> - Assigned permission sets to the user across multiple accounts and logged in via the SSO portal to switch accounts seamlessly | 29/05/2026 | 29/05/2026 | <https://cloudjourney.awsstudygroup.com/> |

### Week 6 Outcomes:

- Solid grasp of IAM policy JSON and able to author **least privilege** policies per group.
- Enabled MFA on root and enforced MFA for the Developers group via `aws:MultiFactorAuthPresent`.
- Configured a strong **password policy** for the account (length, character classes, expiry).
- Created and assumed an **IAM Role** for EC2 instead of hard-coded access keys.
- Used **IAM Access Analyzer** to audit public resources and catch unintended external sharing.
- Set up a basic **IAM Identity Center (SSO)**, created permission sets and assigned them across accounts.

### Difficulties and lessons learned:

- At first I used `Resource: "*"` in a group policy, which was way too broad. After my mentor's reminder I tightened it to specific resources. A common pitfall for beginners.
- An IAM Role for EC2 must be assumed through the metadata service. I tried to embed an access key in user-data; my mentor blocked it and explained why a role is the correct approach.
- IAM Identity Center used to be called AWS Single Sign-On; some older docs still use the old name, so cross-check with the current console is required.