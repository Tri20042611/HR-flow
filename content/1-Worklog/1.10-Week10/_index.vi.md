---
title: "Worklog Tuần 10"
date: 2026-06-22
weight: 10
chapter: false
pre: " <b> 1.10. </b> "
---

### Mục tiêu tuần 10:

- Khởi tạo project **HireFlow AI — Serverless Recruitment Platform** với backend chạy trên AWS Lambda.
- Tìm hiểu **AWS SAM (Serverless Application Model)** và dựng toàn bộ hạ tầng backend bằng template `template.yaml`.
- Vận hành pipeline thử nghiệm: **Upload CV → validate → OCR → LLM score → DynamoDB + SES email**.

> Tuần này có 4 buổi, bỏ Thứ 5. 2 buổi đầu làm quen SAM, 1 buổi migrate sang SAM, buổi cuối test end-to-end.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Tổng quan **AWS SAM**: khác gì với CloudFormation thuần, cú pháp `template.yaml` <br> - Cài **SAM CLI**, chạy `sam init` chọn template Python 3.12 + Lambda <br> - Đọc `template.yaml` mặc định: `AWS::Serverless::Function`, `Events`, `Environment` <br> - Khởi tạo repo `hireflow-sam/` trên máy, cấu trúc thư mục `src/` cho 8 function sau này | 22/06/2026 | 22/06/2026 | <https://docs.aws.amazon.com/serverless-application-model/> |
| 3 | - Học nâng cao: **Globals**, **Layers**, **API event**, **`sam local invoke/start-api`** <br> - Tìm hiểu **SAM parameter** để truyền secrets (LLM API key, SES sender) từ ngoài vào, không hardcode <br> - Dùng `sam local invoke` test thử 1 Lambda "Hello" trước khi viết logic chính <br> - Phân tích kiến trúc HireFlow (8 Lambda + 6 bucket + 2 DDB + SQS + SNS) để lên danh sách resource cho template | 23/06/2026 | 23/06/2026 | <https://docs.aws.amazon.com/serverless-application-model/> |
| 4 | - Viết `template.yaml` cho backend HireFlow: khai báo **6 S3 bucket** (Quarantine, Clean, CV Files, OCR Results, Candidate frontend, HR frontend), **2 DynamoDB table** (`hireflow-jobs`, `hireflow-candidates`) kèm GSI cho SHA-256 dedup, **2 SQS** (`score-queue`, `save-queue`) có DLQ <br> - Khai báo **8 Lambda function**: `presign`, `file-validator`, `extract`, `extract-complete`, `score`, `save-notify`, `candidates`, `jd-management` <br> - Chạy `sam build && sam deploy --guided` deploy stack `hireflow-dev` lên account sandbox | 24/06/2026 | 24/06/2026 | <https://docs.aws.amazon.com/serverless-application-model/> |
| 6 | - Viết Lambda **file-validator** đơn giản (MIME check, size check, SHA-256), bind vào event S3 của bucket Quarantine <br> - Viết Lambda **extract** gọi **Amazon Textract** `StartDocumentTextDetection` cho PDF/DOCX <br> - Test end-to-end 1 CV mẫu: upload → validate → OCR → check kết quả text trong S3 OCR Results <br> - Push code lên Git, ghi `README.md` về cấu trúc repo + cách deploy | 26/06/2026 | 26/06/2026 | <https://docs.aws.amazon.com/textract/> |

### Kết quả đạt được tuần 10:

- Khởi tạo thành công project **HireFlow AI** với repo `hireflow-sam/`, cấu trúc `src/` rõ ràng cho 8 Lambda function.
- Deploy thành công stack SAM `hireflow-dev` gồm **6 bucket, 2 DDB table, 2 SQS + 2 DLQ** lên AWS account.
- Viết được Lambda `file-validator` với MIME magic-byte check, size limit 5MB, SHA-256 dedup qua GSI.
- Tích hợp **Amazon Textract** cho bước OCR CV, xác nhận chạy đúng với PDF mẫu.
- Có pipeline chạy được từ Upload → Validate → Extract, lưu kết quả OCR vào S3.

### Khó khăn và bài học:

- Lúc đầu gắn event S3 cho `file-validator` qua shorthand `Events` của SAM thì gặp **circular dependency** do bucket nằm cùng stack — phải tách trigger bằng Lambda Permission + EventBridge rule tạo thủ công sau khi stack deploy xong.
- `sam local invoke` cần Docker chạy nền, lúc đầu máy chưa bật nên lệnh fail liên tục — bài học phải check Docker trước khi test local.
- Textract `StartDocumentTextDetection` là **async**, không trả text ngay trong response; phải đợi SNS callback `JobStatus=SUCCEEDED` rồi mới gọi `GetDocumentTextDetection` mới có kết quả — phải thiết kế Lambda `extract-complete` riêng để xử lý callback.