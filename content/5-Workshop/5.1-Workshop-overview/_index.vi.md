---
title : "Introduction"
date : 2024-01-01
weight : 1
chapter : false
---

Workshop này hướng dẫn xây dựng đường ống xử lý CV tự động hoàn toàn — từ lúc ứng viên upload CV lên S3, qua các bước xác thực, trích xuất văn bản bằng Amazon Textract, chấm điểm bằng LLM, cho đến khi kết quả được lưu vào DynamoDB và thông báo cho ứng viên.

**Kiến trúc đích:** Queue-based serverless pipeline, tách rời từng giai đoạn bằng SQS để đảm bảo scalability và fault-tolerance. Không có server persistent — toàn bộ Lambda execution, auto-scale theo số lượng CV.

**Tổng quan luồng:**

```
Ứng viên upload CV
    ↓ presigned URL PUT
S3 Quarantine (cách ly file thô)
    ↓ EventBridge trigger
Lambda #2: FileValidator (xác thực MIME, dedup, copy)
    ↓
S3 Clean (file đã xác thực)
    ↓ S3 notification trigger
Lambda #3: Extract (Textract OCR, lưu raw text)
    ↓ EventBridge (Textract complete)
Lambda #3b: ExtractComplete (gộp kết quả Textract)
    ↓ SQS ScoreQueue
Lambda #4: Score (gọi LLM chấm điểm)
    ↓ SQS SaveQueue
Lambda #5: Save&Notify (ghi DynamoDB + gửi email)
    ↓
DynamoDB Candidates ← HR Dashboard đọc qua API Gateway
```
