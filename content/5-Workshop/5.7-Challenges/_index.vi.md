---
title : "Challenges & Future"
date : 2024-01-01
weight : 7
chapter : false
---

### Thách thức đã gặp và cách giải quyết

**Circular dependency giữa S3 bucket và Lambda permission**

SAM không cho phép `BucketName` có tham chiếu đến Lambda ARN trong cùng stack. Giải pháp: dùng Lambda Permission với ARN literal thay vì `!GetAtt`, và tạo EventBridge rule bên ngoài SAM (AWS CLI riêng).

**Textract async timeout**

Textract async job có thể mất 5–10 phút cho file lớn. Giải pháp: dùng EventBridge rule listen Textract completion event thay vì poll trong Lambda. Lambda Extract start job →SNS → EventBridge → Lambda ExtractComplete → SQS. Tách biệt trigger khỏi processing.

**Prompt injection trong CV/JD**

Kẻ xấu có thể chèn chỉ thị vào CV để cố điều khiển LLM. Giải pháp: multi-layer defense: (1) sanitize inputs loại bỏ tag delimiters, (2) dùng sentinel guards `<<<TAG>>>` thay vì `<>` để phân biệt data/instruction, (3) validate + sanitize LLM output trước khi lưu DB.

**LLM response không ổn định**

LLM endpoint có thể timeout hoặc trả JSON không hợp lệ. Giải pháp: retry với exponential backoff (3 lần), output validation + sanitization, fallback sang DLQ nếu fail sau retry.

**Idempotency**

SQS đảm bảo at-least-once delivery — Lambda có thể invoke lại cùng 1 message. Giải pháp: DynamoDB conditional write (`attribute_not_exists`) cho Extract, UpdateItem overwrite cho Score/Save đều idempotent.

### Hướng phát triển tương lai

**Textract synchronous cho file nhỏ**

File < 1MB có thể dùng Textract synchronous (`DetectDocumentText`) để giảm latency từ 2–5 phút xuống < 5 giây. Phân biệt bằng file size hoặc extension.

**Bulk upload**

Hỗ trợ upload nhiều file cùng lúc qua S3 batch operation, xử lý song song hàng trăm CV mà không cần thay đổi Lambda.

**Custom scoring model**

Fine-tune model riêng trên dữ liệu lịch sử ứng viên đã tuyển/thực tập để cải thiện accuracy của scoring.

**Analytics Dashboard**

Bổ sung DynamoDB stream → Lambda → S3 parquet → Athena query để phân tích xu hướng tuyển dụng, thời gian xử lý trung bình, tỷ lệ CV đạt ngưỡng.

**WAF protection**

Thêm AWS WAF cho API Gateway để chống DDoS và SQL injection trên các endpoint HR.

## 9. Cleanup

### Xóa SAM stack

```bash
# Xóa toàn bộ resource (Lambda, S3, DynamoDB, SQS, SNS, DynamoDB, API Gateway)
sam delete --stack-name hireflow
```

SAM sẽ xóa tất cả resource được tạo trong `template.yaml`. **Lưu ý:** S3 buckets cần empty trước khi xóa — SAM sẽ báo lỗi nếu bucket còn object.

```bash
# Nếu SAM delete fail do bucket không empty
aws s3 rb s3://hireflow-quarantine-<ACCOUNT_ID> --force
aws s3 rb s3://hireflow-clean-<ACCOUNT_ID> --force

# Sau đó xóa stack
sam delete --stack-name hireflow
```

### Xóa EventBridge rule (tạo ngoài SAM)

```bash
aws events remove-targets \
  --rule hireflow-quarantine-upload \
  --ids FileValidatorTarget

aws events delete-rule --name hireflow-quarantine-upload
```

### Xóa CloudWatch logs

```bash
# Xóa log groups của các Lambda
aws logs delete-log-group --log-group-name /aws/lambda/hireflow-file-validator
aws logs delete-log-group --log-group-name /aws/lambda/hireflow-extract
aws logs delete-log-group --log-group-name /aws/lambda/hireflow-extract-complete
aws logs delete-log-group --log-group-name /aws/lambda/hireflow-score
aws logs delete-log-group --log-group-name /aws/lambda/hireflow-save-notify

# Hoặc dùng pattern delete tất cả
aws logs describe-log-groups \
  --log-group-name-prefix /aws/lambda/hireflow \
  --query 'logGroups[].logGroupName' \
  --output text | xargs -I {} aws logs delete-log-group --log-group-name {}
```
