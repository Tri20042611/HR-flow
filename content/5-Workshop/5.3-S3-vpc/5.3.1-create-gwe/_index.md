---
title : "EventBridge Rule"
date : 2024-01-01
weight : 3
chapter : false
---

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
