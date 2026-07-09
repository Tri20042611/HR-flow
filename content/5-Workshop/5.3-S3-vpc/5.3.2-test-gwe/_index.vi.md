---
title : "Deploy SAM Stack"
date : 2024-01-01
weight : 3
chapter : false
---

### Step 3: Triển khai SAM stack

```bash
# Validate template trước
sam validate

# Build Lambda layers và dependencies
sam build

# Deploy lần đầu (guided mode)
sam deploy --guided
```

Các tham số cần nhập:

| Parameter | Default | Mô tả |
|---|---|---|
| `Stack Name` | `hireflow` | Tên CloudFormation stack |
| `AWS Region` | `ap-southeast-2` | Region triển khai |
| `AllowedOrigin` | `*` | CORS origin cho API |
| `GeminiApiKeySecret` | `hireflow/gemini-api-key` | Secrets Manager secret name |
| `LLMBaseUrl` | `https://9router.xjanua.me/v1` | LLM endpoint (OpenAI-compatible) |
| `LLMModel` | `Tri_Bui` | Model name |
| `SesFromEmail` | `tuyendung@company.com` | Email gửi thông báo |
| `HRNotificationEmail` | `hr@company.com` | Email nhận alert |

```bash
# Deploy nhanh (sau lần đầu đã lưu config)
sam deploy
```

SAM sẽ tạo:

- 2 S3 bucket (Quarantine + Clean) với SSE-KMS encryption
- 2 DynamoDB table (Candidates + Jobs) với GSI trên `job_id-score_total` và `sha256_hash`
- 2 SQS queue (Score + Save) với DLQ và RedrivePolicy
- 2 SNS topic (HR Alert + Textract Notification)
- 7 Lambda functions
- 1 API Gateway (HTTP API)
- KMS key cho encryption at-rest
- EventBridge rule cho Textract completion
- CloudWatch alarms cho DLQ
