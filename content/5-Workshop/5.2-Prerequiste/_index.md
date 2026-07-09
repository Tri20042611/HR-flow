---
title : "Prerequisites"
date : 2024-01-01
weight : 2
chapter : false
---

- **AWS CLI** configured with IAM credentials (Admin or PowerUser)
- **AWS SAM CLI** version >= 1.90.0: `sam --version`
- **Python 3.12** and **pip**
- **Git** installed
- AWS account with services: S3, Lambda, SQS, SNS, DynamoDB, Textract, Secrets Manager, KMS, EventBridge
- Secrets Manager secret containing LLM API key (default name: `hireflow/gemini-api-key`)

Verify SAM:

```bash
sam --version
# AWS SAM CLI, version 1.90.0 or higher

aws sts get-caller-identity
# Confirm correct account and region
```
