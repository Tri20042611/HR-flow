---
title : "Introduction"
date : 2024-01-01
weight : 1
chapter : false
---

This workshop guides you through building a fully automated CV processing pipeline — from candidate uploading CV to S3, through validation, text extraction via Amazon Textract, LLM-based scoring, to storing results in DynamoDB and notifying the candidate.

**Target architecture:** Queue-based serverless pipeline, separating each stage with SQS for scalability and fault-tolerance. No persistent servers — all Lambda execution, auto-scaling based on CV volume.

**Flow overview:**

```
Candidate uploads CV
    ↓ presigned URL PUT
S3 Quarantine (raw file isolation)
    ↓ EventBridge trigger
Lambda #2: FileValidator (MIME validation, dedup, copy)
    ↓
S3 Clean (validated files)
    ↓ S3 notification trigger
Lambda #3: Extract (Textract OCR, save raw text)
    ↓ EventBridge (Textract complete)
Lambda #3b: ExtractComplete (merge Textract results)
    ↓ SQS ScoreQueue
Lambda #4: Score (LLM scoring)
    ↓ SQS SaveQueue
Lambda #5: Save&Notify (write DynamoDB + send email)
    ↓
DynamoDB Candidates ← HR Dashboard reads via API Gateway
```
