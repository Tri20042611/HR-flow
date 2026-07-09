---
title : "Deploy SAM Stack"
date : 2024-01-01
weight : 3
chapter : false
---

### Step 3: Deploy SAM stack

```bash
# Validate template first
sam validate

# Build Lambda layers and dependencies
sam build

# First-time deployment (guided mode)
sam deploy --guided
```

Parameters to enter:

| Parameter | Default | Description |
|---|---|---|
| `Stack Name` | `hireflow` | CloudFormation stack name |
| `AWS Region` | `ap-southeast-2` | Deployment region |
| `AllowedOrigin` | `*` | CORS origin for API |
| `GeminiApiKeySecret` | `hireflow/gemini-api-key` | Secrets Manager secret name |
| `LLMBaseUrl` | `https://9router.xjanua.me/v1` | LLM endpoint (OpenAI-compatible) |
| `LLMModel` | `Tri_Bui` | Model name |
| `SesFromEmail` | `tuyendung@company.com` | Notification sender email |
| `HRNotificationEmail` | `hr@company.com` | HR alert recipient |

```bash
# Quick deploy (after first run saved config)
sam deploy
```

SAM will create:

- 2 S3 buckets (Quarantine + Clean) with SSE-KMS encryption
- 2 DynamoDB tables (Candidates + Jobs) with GSI on `job_id-score_total` and `sha256_hash`
- 2 SQS queues (Score + Save) with DLQ and RedrivePolicy
- 2 SNS topics (HR Alert + Textract Notification)
- 7 Lambda functions
- 1 API Gateway (HTTP API)
- KMS key for encryption at-rest
- EventBridge rule for Textract completion
- CloudWatch alarms for DLQ
