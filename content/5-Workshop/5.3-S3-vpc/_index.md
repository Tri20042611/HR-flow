---
title : "Project Structure & SAM Deploy"
date : 2024-01-01
weight : 3
chapter : false
---

### Clone and project structure

```bash
git clone <repo-url>
cd hireflow-sam
ls -la
```

Directory structure:

```
hireflow-sam/
├── template.yaml          # SAM template - full IaC
├── src/
│   ├── file_validator/    # Lambda #2
│   ├── extract/           # Lambda #3
│   ├── extract_complete/  # Lambda #3b
│   ├── score/            # Lambda #4
│   ├── save_notify/      # Lambda #5
│   ├── get_presigned_url/
│   ├── jd_management/
│   ├── candidates/
│   └── random_cv/
└── README.md
```

### Step 2: Create EventBridge rule outside SAM

SAM template cannot auto-create EventBridge rule trigger for FileValidator due to circular dependency between S3 bucket and Lambda permission. Rule is created separately via AWS CLI:

```bash
# Create EventBridge rule to catch S3 ObjectCreated on Quarantine bucket
aws events put-rule \
  --name hireflow-quarantine-upload \
  --event-pattern '{
    "source": ["aws.s3"],
    "detail": {
      "bucket": {
        "name": ["hireflow-quarantine-<ACCOUNT_ID>"]
      },
      "object": {
        "key": [{"prefix": "web/"}]
      }
    }
  }' \
  --description "Trigger FileValidator when new file uploaded to S3 Quarantine prefix web/"

# Add Lambda as target
aws events put-targets \
  --rule hireflow-quarantine-upload \
  --targets '[{
    "Id": "FileValidatorTarget",
    "Arn": "arn:aws:lambda:<REGION>:<ACCOUNT_ID>:function:hireflow-file-validator",
    "RetryPolicy": {"MaximumRetryAttempts": 2}
  }]'

# Grant EventBridge permission to invoke Lambda
aws lambda add-permission \
  --function-name hireflow-file-validator \
  --statement-id EventBridgeInvoke \
  --action lambda:InvokeFunction \
  --principal events.amazonaws.com \
  --source-arn "arn:aws:events:<REGION>:<ACCOUNT_ID>:rule/hireflow-quarantine-upload"
```

**Explanation:** EventBridge rule listens for `s3:ObjectCreated:*` events on the `web/` prefix of the Quarantine bucket. When a candidate successfully uploads a CV (file placed at `web/<uuid>.pdf`), the rule triggers the FileValidator Lambda to validate the file.

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
