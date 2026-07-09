---
title : "Project Structure & SAM Deploy"
date : 2024-01-01
weight : 3
chapter : false
---

### Clone và cấu trúc project

```bash
git clone <repo-url>
cd hireflow-sam
ls -la
```

Cấu trúc thư mục:

```
hireflow-sam/
├── template.yaml          # SAM template - toàn bộ IaC
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

### Step 2: Tạo EventBridge rule bên ngoài SAM

SAM template không tự tạo EventBridge rule trigger cho FileValidator vì circular dependency giữa S3 bucket và Lambda permission. Rule được tạo riêng bằng AWS CLI:

```bash
# Tạo EventBridge rule bắt S3 ObjectCreated trên Quarantine bucket
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
  --description "Trigger FileValidator khi có file mới upload vào S3 Quarantine prefix web/"

# Thêm Lambda làm target
aws events put-targets \
  --rule hireflow-quarantine-upload \
  --targets '[{
    "Id": "FileValidatorTarget",
    "Arn": "arn:aws:lambda:<REGION>:<ACCOUNT_ID>:function:hireflow-file-validator",
    "RetryPolicy": {"MaximumRetryAttempts": 2}
  }]'

# Cấp quyền EventBridge invoke Lambda
aws lambda add-permission \
  --function-name hireflow-file-validator \
  --statement-id EventBridgeInvoke \
  --action lambda:InvokeFunction \
  --principal events.amazonaws.com \
  --source-arn "arn:aws:events:<REGION>:<ACCOUNT_ID>:rule/hireflow-quarantine-upload"
```

**Giải thích:** EventBridge rule lắng nghe event `s3:ObjectCreated:*` trên prefix `web/` của Quarantine bucket. Khi ứng viên upload CV thành công (file đặt tại `web/<uuid>.pdf`), rule trigger Lambda FileValidator để kiểm tra file.

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
