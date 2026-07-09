---
title : "Validation & Analysis"
date : 2024-01-01
weight : 6
chapter : false
---

### End-to-end test

**Upload a real CV and monitor the pipeline:**

```bash
# 1. Get presigned URL (or upload directly if frontend is ready)
# 2. Upload test CV to Quarantine bucket
aws s3 cp test-cv.pdf s3://hireflow-quarantine-<ACCOUNT_ID>/web/test-cv.pdf

# 3. Monitor FileValidator Lambda logs
aws logs tail /aws/lambda/hireflow-file-validator --follow --format short

# 4. Check file copied to Clean bucket
aws s3 ls s3://hireflow-clean-<ACCOUNT_ID>/

# 5. Monitor Extract Lambda logs
aws logs tail /aws/lambda/hireflow-extract --follow --format short

# 6. Check OCR text saved
aws s3 ls s3://hireflow-clean-<ACCOUNT_ID>/ocr/

# 7. Monitor Score Lambda logs
aws logs tail /aws/lambda/hireflow-score --follow --format short

# 8. Check results in DynamoDB
aws dynamodb scan \
  --table-name hireflow-candidates \
  --filter-expression "score_total > :zero" \
  --expression-attribute-values '{"N": {"N": "0"}}' \
  --output json
```

**Expected output after pipeline completes:**

```json
{
  "candidate_id": "uuid-...",
  "job_id": "job-001",
  "score_total": 75.5,
  "score_detail": {
    "experience": 80.0,
    "skills": 70.0,
    "education": 85.0,
    "soft_skill": 70.0
  },
  "summary": "Candidate has 5 years backend experience, proficient in Python and AWS. ...",
  "flags": [],
  "status": "reviewing",
  "confidence": "high"
}
```

### CloudWatch Metrics analysis

Check pipeline performance:

```bash
# Lambda invocation count and error rate
aws cloudwatch get-metric-statistics \
  --namespace AWS/Lambda \
  --metric-name Invocations \
  --dimensions Name=FunctionName,Value=hireflow-file-validator \
  --start-time $(date -u -d '1 hour ago' +%Y-%m-%dT%H:%M:%S) \
  --end-time $(date -u +%Y-%m-%dT%H:%M:%S) \
  --period 300 \
  --statistics Sum

# SQS queue depth (messages waiting)
aws cloudwatch get-metric-statistics \
  --namespace AWS/SQS \
  --metric-name ApproximateNumberOfMessagesVisible \
  --dimensions Name=QueueName,Value=hireflow-score-queue \
  --start-time $(date -u -d '1 hour ago' +%Y-%m-%dT%H:%M:%S) \
  --end-time $(date -u +%Y-%m-%dT%H:%M:%S) \
  --period 300 \
  --statistics Average
```

### DLQ inspection (dead-letter queue)

Check for failed messages:

```bash
# Number of messages in DLQ
aws sqs get-queue-attributes \
  --queue-url https://sqs.ap-southeast-2.amazonaws.com/<ACCOUNT_ID>/hireflow-score-dlq \
  --attribute-names ApproximateNumberOfMessages

# Read message from DLQ for debugging
aws sqs receive-message \
  --queue-url https://sqs.ap-southeast-2.amazonaws.com/<ACCOUNT_ID>/hireflow-score-dlq \
  --max-number-of-messages 1

# CloudWatch alarm is configured — HR will receive email if DLQ has messages
```
