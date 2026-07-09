---
title: "Week 10 Worklog"
date: 2026-06-22
weight: 10
chapter: false
pre: " <b> 1.10. </b> "
---

### Week 10 Objectives:

- Initialize the **HireFlow AI — Serverless Recruitment Platform** project with backend running on AWS Lambda.
- Learn **AWS SAM (Serverless Application Model)** and set up the entire backend infrastructure using `template.yaml`.
- Run the end-to-end experimental pipeline: **Upload CV → validate → OCR → LLM score → DynamoDB + SES email**.

> This week has 4 sessions (skipping Thursday). The first 2 sessions cover SAM familiarization, 1 session for SAM migration, and the final session for end-to-end testing.

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| Mon | - **AWS SAM** overview: how it differs from vanilla CloudFormation, `template.yaml` syntax <br> - Install **SAM CLI**, run `sam init` selecting Python 3.12 + Lambda template <br> - Read the default `template.yaml`: `AWS::Serverless::Function`, `Events`, `Environment` <br> - Initialize repo `hireflow-sam/` locally, `src/` directory structure for 8 functions to be added later | 22/06/2026 | 22/06/2026 | <https://docs.aws.amazon.com/serverless-application-model/> |
| Tue | - Advanced topics: **Globals**, **Layers**, **API event**, **`sam local invoke/start-api`** <br> - Learn **SAM parameters** to pass secrets (LLM API key, SES sender) externally, avoiding hardcoding <br> - Use `sam local invoke` to test a "Hello" Lambda before writing the main logic <br> - Analyze HireFlow architecture (8 Lambda + 6 buckets + 2 DDB + SQS + SNS) to compile the resource list for the template | 23/06/2026 | 23/06/2026 | <https://docs.aws.amazon.com/serverless-application-model/> |
| Wed | - Write `template.yaml` for HireFlow backend: declare **6 S3 buckets** (Quarantine, Clean, CV Files, OCR Results, Candidate frontend, HR frontend), **2 DynamoDB tables** (`hireflow-jobs`, `hireflow-candidates`) with GSI for SHA-256 dedup, **2 SQS** (`score-queue`, `save-queue`) with DLQ <br> - Declare **8 Lambda functions**: `presign`, `file-validator`, `extract`, `extract-complete`, `score`, `save-notify`, `candidates`, `jd-management` <br> - Run `sam build && sam deploy --guided` to deploy stack `hireflow-dev` to the sandbox account | 24/06/2026 | 24/06/2026 | <https://docs.aws.amazon.com/serverless-application-model/> |
| Fri | - Write simple Lambda **file-validator** (MIME check, size check, SHA-256), bind to S3 event on the Quarantine bucket <br> - Write Lambda **extract** calling **Amazon Textract** `StartDocumentTextDetection` for PDF/DOCX <br> - End-to-end test with 1 sample CV: upload → validate → OCR → check text result in S3 OCR Results bucket <br> - Push code to Git, write `README.md` covering repo structure and deployment steps | 26/06/2026 | 26/06/2026 | <https://docs.aws.amazon.com/textract/> |

### Week 10 Achievements:

- Successfully initialized **HireFlow AI** project with repo `hireflow-sam/`, clean `src/` directory structure for 8 Lambda functions.
- Successfully deployed SAM stack `hireflow-dev` comprising **6 buckets, 2 DDB tables, 2 SQS + 2 DLQ** to AWS account.
- Built Lambda `file-validator` with MIME magic-byte check, 5MB size limit, SHA-256 dedup via GSI.
- Integrated **Amazon Textract** for CV OCR step, confirmed working correctly with a sample PDF.
- Pipeline runs from Upload → Validate → Extract, storing OCR results in S3.

### Challenges and Lessons Learned:

- Initially attaching S3 event to `file-validator` via SAM shorthand `Events` caused **circular dependency** because the bucket is in the same stack — had to split the trigger using Lambda Permission + EventBridge rule created manually after stack deployment.
- `sam local invoke` requires Docker running in the background; the machine did not have Docker enabled initially so the command kept failing — lesson: check Docker before running local tests.
- Textract `StartDocumentTextDetection` is **async**; it does not return text immediately in the response. Had to wait for SNS callback `JobStatus=SUCCEEDED` then call `GetDocumentTextDetection` for the actual result — needed to design a separate Lambda `extract-complete` to handle the callback.
