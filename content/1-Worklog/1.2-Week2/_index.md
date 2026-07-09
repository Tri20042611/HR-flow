---
title: "Worklog Week 2"
date: 2026-04-27
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---

### Week 2 Goals:

- Deep dive into EC2: instance families, AMI, EBS, Metadata Service and how to use Session Manager securely without opening SSH port.
- Practice basic Linux operations on EC2 (users, permissions, processes, logs).

> Light schedule this week, only 2 real sessions because mid-week I had to focus on university assignments. The rest of the time I read docs at home.

### Tasks done this week:

| Session | Date | Task | Reference |
| --- | --- | --- | --- |
| Session 1 | 28/04/2026 | Deep dive into EC2 & EBS: <br> - Instance families: general (t3, m5), compute optimized (c5), memory optimized (r5) <br> - IMDSv1 vs IMDSv2 and SSRF risks <br> - EBS volume types (gp3, io2, st1, sc1) <br> - Hands-on: created an EBS volume, attached to EC2, formatted ext4 and mounted at `/data` | <https://cloudjourney.awsstudygroup.com/> |
| Session 2 | 30/04/2026 | Session Manager & Linux basics: <br> - Attached IAM role with SSM permissions to EC2, verified SSM Agent <br> - Closed port 22 on Security Group, connected via Session Manager from console & CLI <br> - Reviewed Linux: `useradd`, `chmod`, `chown`, `systemctl`, `journalctl`, `ss`, `df`, `du` <br> - Read `/var/log/` to debug a Nginx start failure | <https://cloudjourney.awsstudygroup.com/> |

### Week 2 Outcomes:

- Differentiated EC2 instance families and chose the right one for each task.
- Understood **IMDSv1 vs IMDSv2** and the SSRF risk of leaving IMDSv1 on.
- Created, attached, mounted, snapshotted and resized EBS volumes confidently.
- Successfully configured **Session Manager** to access EC2 without opening port 22 — a best practice I want to keep using.
- More confident reading and debugging Linux system logs.

### Difficulties and lessons learned:

- Session Manager did not connect at first because I forgot to attach the IAM role with `AmazonSSMManagedInstanceCore` to the EC2. After attaching the role, it worked.
- When mounting EBS for the first time I forgot to run `mkfs.ext4` before `mount`, got a `wrong fs type` error.
- Was confused between `systemctl` and `service`; decided to just stick with `systemctl` for modern services.