---
title: "Week 12 Worklog"
date: 2026-07-06
weight: 12
chapter: false
pre: " <b> 1.12. </b> "
---

### Week 12 Objectives:

- Complete the **frontend SPA** (candidate app + HR dashboard) hosted on S3, ready for end-to-end demo via cloudflared tunnel.
- Run a **full HireFlow system demo** from CV upload to HR viewing the candidate ranking; record video/screenshots for the report.
- Write the **final worklog report** summarizing 12 weeks of internship and the HireFlow project.

> Final week of the program, 4 sessions. First 2 sessions focus on product (frontend + demo); last 2 sessions are for writing the summary worklog report.

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| Mon | - Build **Candidate SPA** (`candidate-app/`): CV upload form, call API `GET /apply/presigned-url` for URL, upload directly to S3 Quarantine <br> - Display realtime apply status (poll API `GET /candidates/{id}`) <br> - Build **HR dashboard SPA**: candidate list table, filter by `job_id`, sort by `score`, view CV details <br> - Host both SPAs on **S3 static website**: `apply-public-read`, set `index.html` + CORS | 06/07/2026 | 06/07/2026 | <https://docs.aws.amazon.com/AmazonS3/latest/userguide/WebsiteHosting.html> |
| Tue | - End-to-end HireFlow demo: create 1 sample JD via API, upload 3–5 test CVs from `cv-test/` <br> - Observe pipeline on CloudWatch Logs, confirm score and email reach the correct candidate <br> - Use **cloudflared tunnel** to point domain `hireflow-dev` to S3 static, demo to mentor <br> - Record **screenshots** of each step: upload form, HR dashboard ranking, email inbox, CloudWatch logs | 07/07/2026 | 07/07/2026 | <https://docs.aws.amazon.com/AmazonCloudWatch/> |
| Wed | - Write **final worklog report**: summarize 12 weeks, HireFlow architecture, achievements, lessons learned, future roadmap <br> - Compile worklogs from weeks 1–12 into PDF with illustrations <br> - Prepare **presentation slides** (10–12 slides) for the final internship presentation <br> - Review repo README.md to ensure onboarding is clear for newcomers | 08/07/2026 | 08/07/2026 | <https://cloudjourney.awsstudygroup.com/> |
| Fri | - **Final internship presentation** with mentor and fellow program members <br> - Present HireFlow architecture, demo live on cloudflared tunnel <br> - Receive mentor feedback on: CORS/Auth trade-offs, third-party LLM endpoint, Cognito scope <br> - Clean up AWS sandbox resources (delete stack `hireflow-dev` and test resources) <br> - Note improvements for future: re-enable GuardDuty, structured resume parsing, batch scoring, multi-channel ingest | 10/07/2026 | 10/07/2026 | <https://cloudjourney.awsstudygroup.com/> |

### Week 12 Achievements:

- Completed **2 SPA frontends** (candidate + HR dashboard) hosted on S3 static website, accessible via cloudflared tunnel.
- Successfully demonstrated **end-to-end HireFlow**: upload 3–5 CVs → validate → OCR → LLM score → DynamoDB ranking → confirmation email → HR views candidate table.
- Wrote and submitted **final worklog report** summarizing 12 weeks with illustrations from each phase.
- Presented and defended the project before mentor; received positive feedback on the SQS-based decoupled architecture and SAM IaC infrastructure.
- Successfully completed the **First Cloud AI Journey internship**, with a demo-ready product and full worklog for future reference.

### Challenges and Lessons Learned:

- CloudFront was not enabled due to cost; used **cloudflared tunnel** for dev demo — documented as an acceptable trade-off given low CV traffic and internal-only use.
- During email demo, some test candidates' emails landed in the **spam folder** because SPF/DKIM was not properly configured — documented under "Known limitations"; production needs custom domain + DKIM setup.
- The final worklog report took considerable time to compile because 12 weeks of information needed to be synthesized — lesson: write concise weekly logs so final compilation only requires editing, not rewriting.
- End-of-program cleanup is easy to forget: SAM stack deletion left Secrets Manager secrets and S3 buckets with versioning still holding objects — had to use `scripts/cleanup.sh` to remove everything, including versioned objects.
