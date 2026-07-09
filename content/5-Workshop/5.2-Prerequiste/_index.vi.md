---
title : "Prerequisites"
date : 2024-01-01
weight : 2
chapter : false
---

- **AWS CLI** đã configure với credentials có quyền IAM phù hợp (Admin hoặc PowerUser)
- **AWS SAM CLI** phiên bản >= 1.90.0: `sam --version`
- **Python 3.12** và **pip**
- **Git** đã cài đặt
- Tài khoản AWS với các dịch vụ: S3, Lambda, SQS, SNS, DynamoDB, Textract, Secrets Manager, KMS, EventBridge
- Secrets Manager đã tạo secret chứa LLM API key (tên mặc định: `hireflow/gemini-api-key`)

Kiểm tra SAM:

```bash
sam --version
# AWS SAM CLI, version 1.90.0 or higher

aws sts get-caller-identity
# Xác nhận đúng account và region
```
