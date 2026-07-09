---
title : "EventBridge Rule"
date : 2024-01-01
weight : 3
chapter : false
---

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
