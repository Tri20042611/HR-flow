---
title : "Validation & Analysis"
date : 2024-01-01
weight : 6
chapter : false
---

## 7. Validation & Analysis

### 7.1 End-to-end test

**Upload 1 CV thật và theo dõi pipeline:**

```bash
# 1. Lấy presigned URL (hoặc upload trực tiếp nếu có frontend)
# 2. Upload test CV lên Quarantine bucket
aws s3 cp test-cv.pdf s3://hireflow-quarantine-<ACCOUNT_ID>/web/test-cv.pdf

# 3. Theo dõi Lambda FileValidator logs
aws logs tail /aws/lambda/hireflow-file-validator --follow --format short

# 4. Kiểm tra file đã copy sang Clean bucket
aws s3 ls s3://hireflow-clean-<ACCOUNT_ID>/

# 5. Theo dõi Lambda Extract logs
aws logs tail /aws/lambda/hireflow-extract --follow --format short

# 6. Kiểm tra OCR text đã lưu
aws s3 ls s3://hireflow-clean-<ACCOUNT_ID>/ocr/

# 7. Theo dõi Lambda Score logs
aws logs tail /aws/lambda/hireflow-score --follow --format short

# 8. Kiểm tra kết quả trong DynamoDB
aws dynamodb scan \
  --table-name hireflow-candidates \
  --filter-expression "score_total > :zero" \
  --expression-attribute-values '{"N": {"N": "0"}}' \
  --output json
```

**Expected output sau khi pipeline chạy xong:**

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
  "summary": "Ứng viên có 5 năm kinh nghiệm backend, thành thạo Python và AWS. ...",
  "flags": [],
  "status": "reviewing",
  "confidence": "high"
}
```

### 7.2 CloudWatch Metrics phân tích

Kiểm tra pipeline performance:

```bash
# Lambda invocation count và error rate
aws cloudwatch get-metric-statistics \
  --namespace AWS/Lambda \
  --metric-name Invocations \
  --dimensions Name=FunctionName,Value=hireflow-file-validator \
  --start-time $(date -u -d '1 hour ago' +%Y-%m-%dT%H:%M:%S) \
  --end-time $(date -u +%Y-%m-%dT%H:%M:%S) \
  --period 300 \
  --statistics Sum

# SQS queue depth (message chờ xử lý)
aws cloudwatch get-metric-statistics \
  --namespace AWS/SQS \
  --metric-name ApproximateNumberOfMessagesVisible \
  --dimensions Name=QueueName,Value=hireflow-score-queue \
  --start-time $(date -u -d '1 hour ago' +%Y-%m-%dT%H:%M:%S) \
  --end-time $(date -u +%Y-%m-%dT%H:%M:%S) \
  --period 300 \
  --statistics Average
```

### 7.3 DLQ inspection (dead-letter queue)

Kiểm tra nếu có message thất bại:

```bash
# Số message trong DLQ
aws sqs get-queue-attributes \
  --queue-url https://sqs.ap-southeast-2.amazonaws.com/<ACCOUNT_ID>/hireflow-score-dlq \
  --attribute-names ApproximateNumberOfMessages

# Đọc message từ DLQ để debug
aws sqs receive-message \
  --queue-url https://sqs.ap-southeast-2.amazonaws.com/<ACCOUNT_ID>/hireflow-score-dlq \
  --max-number-of-messages 1

# CloudWatch alarm đã configured — HR sẽ nhận email nếu DLQ có message
```
