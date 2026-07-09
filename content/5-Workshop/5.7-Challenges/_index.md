---
title : "Challenges & Future"
date : 2024-01-01
weight : 7
chapter : false
---

### Challenges encountered and solutions

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

### Future development directions

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

## 9. Cleanup

### Delete SAM stack

```bash
# Delete all resources (Lambda, S3, DynamoDB, SQS, SNS, DynamoDB, API Gateway)
sam delete --stack-name hireflow
```

SAM will delete all resources created in `template.yaml`. **Note:** S3 buckets must be empty before deletion — SAM will error if bucket still has objects.

```bash
# If SAM delete fails due to non-empty bucket
aws s3 rb s3://hireflow-quarantine-<ACCOUNT_ID> --force
aws s3 rb s3://hireflow-clean-<ACCOUNT_ID> --force

# Then delete stack
sam delete --stack-name hireflow
```

### Delete EventBridge rule (created outside SAM)

```bash
aws events remove-targets \
  --rule hireflow-quarantine-upload \
  --ids FileValidatorTarget

aws events delete-rule --name hireflow-quarantine-upload
```

### Delete CloudWatch logs

```bash
# Delete Lambda log groups
aws logs delete-log-group --log-group-name /aws/lambda/hireflow-file-validator
aws logs delete-log-group --log-group-name /aws/lambda/hireflow-extract
aws logs delete-log-group --log-group-name /aws/lambda/hireflow-extract-complete
aws logs delete-log-group --log-group-name /aws/lambda/hireflow-score
aws logs delete-log-group --log-group-name /aws/lambda/hireflow-save-notify

# Or use pattern to delete all
aws logs describe-log-groups \
  --log-group-name-prefix /aws/lambda/hireflow \
  --query 'logGroups[].logGroupName' \
  --output text | xargs -I {} aws logs delete-log-group --log-group-name {}
```
