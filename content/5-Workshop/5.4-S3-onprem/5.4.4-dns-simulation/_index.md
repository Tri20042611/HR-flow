---
title : "Challenges & Future"
date : 2024-01-01
weight : 7
chapter : false
---

## 8. Challenges & Future Direction

### 8.1 Challenges encountered and solutions

**Circular dependency between S3 bucket and Lambda permission**

SAM doesn't allow `BucketName` referencing Lambda ARN in the same stack. Solution: use Lambda Permission with literal ARN instead of `!GetAtt`, and create EventBridge rule outside SAM (separate AWS CLI).

**Textract async timeout**

Textract async job can take 5–10 minutes for large files. Solution: use EventBridge rule to listen Textract completion event instead of polling in Lambda. Lambda Extract starts job →SNS → EventBridge → Lambda ExtractComplete → SQS. Separates trigger from processing.

**Prompt injection in CV/JD**

Bad actors can insert instructions in CV to try controlling the LLM. Solution: multi-layer defense: (1) sanitize inputs to remove tag delimiters, (2) use sentinel guards `<<<TAG>>>` instead of `<>` to distinguish data/instruction, (3) validate + sanitize LLM output before saving to DB.

**Unstable LLM responses**

LLM endpoint may timeout or return invalid JSON. Solution: retry with exponential backoff (3 times), output validation + sanitization, fallback to DLQ if fail after retry.

**Idempotency**

SQS ensures at-least-once delivery — Lambda may re-invoke the same message. Solution: DynamoDB conditional write (`attribute_not_exists`) for Extract, UpdateItem overwrite for Score/Save are all idempotent.

### 8.2 Future development directions

**Textract synchronous for small files**

Files < 1MB can use Textract synchronous (`DetectDocumentText`) to reduce latency from 2–5 minutes to < 5 seconds. Distinguish by file size or extension.

**Bulk upload**

Support uploading multiple files at once via S3 batch operation, processing hundreds of CVs in parallel without changing Lambda.

**Custom scoring model**

Fine-tune private model on historical data of hired/interned candidates to improve scoring accuracy.

**Analytics Dashboard**

Add DynamoDB stream → Lambda → S3 parquet → Athena query for recruitment trend analysis, average processing time, CV pass rate.

**WAF protection**

Add AWS WAF for API Gateway to prevent DDoS and SQL injection on HR endpoints.
