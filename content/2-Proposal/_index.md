---
title: "HireFlow AI — Serverless Recruitment Platform"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

### 1. Executive Summary

HireFlow AI is an automated candidate CV screening platform for HR departments, replacing repetitive manual tasks with an AWS Serverless platform integrated with artificial intelligence. The system supports CV intake from multiple channels (web form, email, direct upload), automatically extracts text via Amazon Textract, scores candidates using a large language model (LLM), and presents a ranked list on an HR Dashboard. The platform handles up to 10–15 open job postings and processes up to 200 CVs/day simultaneously, with scalability to match actual demand.

### 2. Problem Statement

**Current Problem**

Current recruitment relies on manual operations: receiving CVs via email, downloading and reading each file, copying information to spreadsheets, cross-referencing with job descriptions (JD), and manually scoring candidates. Each CV takes 8–12 minutes to process; with 100 applications, an HR staff member needs approximately 15–20 hours of continuous work just for initial screening. Evaluations are inconsistent across candidates, potential resumes are easily overlooked, and there is no audit trail.

**Solution**

The platform uses Amazon S3 as a quarantine storage zone to receive raw CVs, AWS Lambda with Amazon Textract to extract text from PDF and DOCX, Amazon SQS to decouple processing stages, AWS Lambda with a large language model (LLM) to score candidates against the JD, and Amazon DynamoDB to store structured data. The React HR Dashboard is hosted on Amazon S3 distributed via CloudFront and secured with Amazon Cognito. The batch processing pipeline is fully automated, returning results within 2–3 minutes per CV.

**Benefits and ROI**

The solution cuts initial screening time by 80% (from 10 minutes to under 2 minutes per CV), ensures consistent evaluation via unified LLM scoring criteria, and creates a complete history for future auditing and analysis. Infrastructure cost is estimated under $5 USD/month for 200 CVs/day (per AWS Pricing Calculator), totalling under $60 USD/year. The break-even period is estimated at under 6 months based on saved HR man-hours.

### 3. Solution Architecture

**Overview**

HireFlow AI adopts a queue-based AWS Serverless architecture to ensure scalability, fault tolerance, and decoupling between stages. Candidates upload CVs via a presigned URL directly to the S3 Quarantine bucket, bypassing API Gateway to reduce load and exceed the 6 MB payload limit. After passing the format validation layer, CVs are moved to the S3 Clean bucket and queued into the extraction pipeline.

![HireFlow AI Architecture](/images/2-Proposal/hireflow_architecture.png)

**AWS Services Used**

- **Amazon S3 (2 buckets):** Quarantine zone receives raw files; Clean zone stores validated files for processing.
- **AWS Lambda (7 functions):** File-validator, Extract, Extract-complete, Score, Save-and-notify, Candidates, JD-management, Get-presigned-url.
- **Amazon API Gateway:** HTTP API connecting React SPA to Lambda, with Cognito authorizer.
- **Amazon SQS (2 queues):** Score-queue buffers before scoring; Save-queue buffers before saving and notifying.
- **Amazon Textract:** Extracts text and table structure from CV PDF/DOCX.
- **Amazon DynamoDB (2 tables):** Candidates stores scored profiles; Jobs stores job postings + JD + scoring criteria.
- **Amazon Cognito:** User pool for HR (5 accounts), separate pool for candidates (optional).
- **Amazon SNS:** Textract notification topic, HR alert topic for pipeline failure emails.
- **Amazon CloudFront + S3 Hosting:** Distributes the HR Dashboard (React SPA).
- **AWS Secrets Manager:** Stores LLM provider API key.
- **AWS KMS:** Encrypts data in S3, DynamoDB, and SQS.

**Component Design**

- **Zone 1 — Intake Layer:** Candidate requests a presigned URL from Lambda `get-presigned-url`, then uploads the file directly to S3 Quarantine via HTTPS PUT.
- **Zone 2 — Security Layer:** EventBridge rule catches S3 ObjectCreated events at the `web/` prefix; Lambda `file-validator` checks MIME (magic bytes), size ≤ 5 MB, SHA-256 dedup, then copies to Clean and deletes from Quarantine.
- **Zone 3 — Processing Layer:** Lambda `extract` reads the file from Clean, calls Textract async, stores JobId; SNS Textract calls Lambda `extract-complete` to merge results, sends SQS message to the scoring queue.
- **AI Layer:** Lambda `score` receives SQS message, reads JD from DynamoDB, calls LLM (OpenAI-compatible endpoint), saves score and reasoning to DynamoDB, sends SQS message to the save queue.
- **Persistence & Notification Layer:** Lambda `save-and-notify` writes the final record to DynamoDB and sends a confirmation email to the candidate via SES.
- **Interface Layer:** HR Dashboard React SPA calls API Gateway → Lambda `candidates`/`jd-management` to read/write DynamoDB, protected by Cognito.

### 4. Technical Implementation

**Implementation Phases**

The project has 4 main phases, each with clear deliverables:

1. **Research and Architecture Design (Weeks 1–2):** Survey current recruitment process, evaluate suitable AWS services, draft overall architecture diagram and sequence diagrams for each main flow.
2. **Cost Estimation and Feasibility Assessment (Week 3):** Use AWS Pricing Calculator to estimate costs for low/average/high load scenarios; identify potential bottlenecks and cost-reduction strategies (Lambda memory sizing, S3 lifecycle, SQS batching).
3. **Architecture Optimization (Weeks 4–5):** Refine design — reduce Lambda load by splitting the pipeline into multiple SQS-staged steps, restructure IAM roles following least-privilege principle, add retry mechanisms and dead-letter queues.
4. **Development, Testing, and Deployment (Weeks 6–12):** Program Lambda in Python 3.12 with boto3, use AWS SAM as IaC to define all infrastructure; test each Lambda with sample events; run end-to-end on a real account with 10 sample CVs.

**Technical Requirements**

- **AWS SAM:** Define the entire stack (Lambda, S3, DynamoDB, SQS, SNS, Cognito, API Gateway, CloudFront) in `template.yaml`.
- **AWS Lambda:** Python 3.12, boto3 SDK, structured logging, X-Ray tracing.
- **Amazon Textract:** `StartDocumentAnalysis` with `TABLES` + `FORMS` features, polling or SNS callback for async completion.
- **LLM Integration:** OpenAI-compatible endpoint (pre-provisioned via 9router.xjanua.me), key stored in Secrets Manager, prompt engineering for 4-criteria scoring (skills, experience, education, fit).
- **Frontend:** React 19 + Vite + Amazon Cognito Identity SDK, build artifact deployed to S3 distributed via CloudFront.
- **Security:** IAM least-privilege, S3 block public access, S3 + DynamoDB + SQS encryption with KMS, VPC endpoints optional but network ACLs reviewed.
- **Observability:** CloudWatch Logs (7-day retention), CloudWatch Metrics (Lambda duration + error count), CloudWatch Alarm when Lambda error rate > 5%.

### 5. Timeline & Milestones

Project runs across 12 weeks aligned with the internship period.

### 6. Budget Estimation

**Infrastructure Costs (estimated for 200 CVs/day)**

| Category | Service | Est. Cost |
|---|---|---|
| Storage & CDN | S3 (2 buckets) | ~$2.50/month |
| Storage & CDN | CloudFront + S3 Hosting | ~$1.00/month |
| Compute | Lambda (7 functions) | ~$5.00/month |
| AI/ML | Textract | ~$15.00/month |
| Networking | API Gateway (HTTP API) | ~$3.50/month |
| Networking | SQS (2 queues) | ~$0.00/month |
| Networking | SNS (2 topics) | ~$0.00/month |
| Database | DynamoDB (2 tables) | ~$8.00/month |
| Security | Secrets Manager | ~$0.40/month |
| Security | KMS | ~$1.00/month |
| Monitoring | CloudWatch | ~$2.00/month |
| **Total** | | **~$38.40/month** |

**Key Notes:**
1. Textract is the dominant cost at $0.015/page — scales linearly with CV volume.
2. SQS, SNS, and Cognito are all within free tier limits at expected volumes.
3. Lambda has 1M free requests + 400K GB-seconds free per month.
4. Costs grow mainly in Textract and Lambda as CV volume increases.

**Hardware:** None — all processing is in the cloud; candidates use a web browser.

### 7. Risk Assessment

**Risk Matrix**

| Risk | Impact | Probability |
|---|---|---|
| Textract quota exceeds free tier | Medium | Low |
| LLM API timeout/down | High | Low |
| Cost overrun due to Textract | Medium | Medium |
| Malicious CV / malware injection | High | Low |
| Race condition on concurrent CVs | Medium | Low |

**Mitigation Strategies**
- **Quota:** Enable CloudWatch alarm on Textract `ThrottledCount`, request quota raise when needed.
- **LLM availability:** Implement circuit breaker in Lambda `score`; SQS queue retains messages for up to 3 retries before moving to DLQ.
- **Secret hygiene:** Access key via environment variables from Secrets Manager, never log it; rotate key every 90 days.
- **Cost:** AWS Billing budget alert at $5/month; auto-disable Textract if exceeded.
- **Malware:** Pipeline validates MIME via magic bytes (does not trust Content-Type header); CVs are kept in Quarantine for 7 days before deletion for HR review.
- **Concurrency:** DynamoDB conditional write on SHA-256 hash to prevent duplicates; idempotency key on each message.

**Contingency Plan**
- Revert to manual process if system is down for more than 24 hours.
- All infrastructure is IaC via SAM — recovery in under 30 minutes with `sam deploy`.
- Processed CVs are retained in the Clean bucket for 90 days as backup.

### 8. Expected Outcomes

- **Technical Improvements:** 80% reduction in CV screening time; consistent LLM-based scoring; full audit trail; scalable to 500 CVs/day without significant cost increase.
- **Long-term Value:** Historical candidate data in DynamoDB can be used for recruitment trend analysis and training proprietary scoring models; the platform can be repurposed for similar screening problems (application approvals, vendor assessments).
