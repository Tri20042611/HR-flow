---
title: "Blog 3"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.3. </b> "
---

# Tự động hóa trích xuất và phân tích Audit Log PostgreSQL với Amazon S3

Trong quá trình tìm hiểu về AWS, mình đọc được một bài về việc tự động hóa pipeline xử lý audit log cho Amazon RDS PostgreSQL và Aurora PostgreSQL.

## Bối cảnh

Khi bật pgAudit trên PostgreSQL, toàn bộ hoạt động SQL như SELECT, INSERT, UPDATE, DELETE, DDL đều được ghi lại. Tuy nhiên, log này nằm trong CloudWatch Logs dưới dạng text thô, không có cấu trúc. Việc xem xét hay phân tích thủ công tốn rất nhiều thời gian, đặc biệt với các đội ngũ cần đáp ứng yêu cầu bảo mật và kiểm toán thường xuyên.

## Giải pháp

Một pipeline tự động hoàn toàn dựa trên kiến trúc event-driven gồm các thành phần sau:

1. PostgreSQL sinh audit log qua pgAudit
2. CloudWatch Logs nhận log và dùng subscription filter để phát hiện các dòng chứa từ khóa "AUDIT"
3. Kích hoạt AWS Lambda xử lý
4. Lambda parse text thô thành CSV có cấu trúc với đầy đủ các trường: timestamp, user, database, action, object_type, object_name, query
5. Lưu vào Amazon S3 theo cấu trúc phân vùng ngày tháng năm
6. Tùy chọn dùng Amazon Athena để query trực tiếp bằng SQL

![pgAudit Pipeline Architecture](/images/3-BlogsPosted/3.3-Blog3/pgaudit_pipeline.png)

Toàn bộ hạ tầng được đóng gói trong một CloudFormation template, tạo S3 bucket có mã hóa và versioning, IAM role least-privilege, Lambda function Python 3.12, và subscription filter. Thời gian deploy khoảng 2–3 phút.

## Về chi phí

Phần chi phí đáng chú ý nhất là **CloudWatch Logs ingestion ở mức $0.50/GB**, vì nó tỷ lệ thuận trực tiếp với số lượng câu lệnh SQL được audit. Lambda và S3 thường không phải lo vì CloudWatch subscription filter gom nhiều log event vào một lần invoke, nên chi phí hai phần này rất thấp.

S3 lifecycle policy tự động chuyển log cũ sang Standard-IA sau 90 ngày và Glacier sau 1 năm để tối ưu chi phí lưu trữ dài hạn.

Nếu không cần audit toàn bộ câu lệnh, có thể đổi cấu hình `pgaudit.log = 'ddl, role'` thay vì `all` để giảm đáng kể volume log.

## Lưu ý bảo mật

Vì pipeline lưu lại toàn bộ nội dung câu SQL bao gồm cả tham số, nên có khả năng chứa dữ liệu nhạy cảm như thông tin nhận dạng cá nhân (PII). Khuyến nghị:

- Giới hạn quyền truy cập S3 bucket
- Cân nhắc dùng Amazon Macie để scan dữ liệu nhạy cảm
- Có thể mở rộng Lambda để mask hoặc redact các tham số trước khi ghi xuống S3

## Kết luận

Đây là một giải pháp thực tế giải quyết đúng vấn đề nhiều team đang gặp: log thì có nhưng không dùng được hiệu quả. Pipeline này chuyển audit log từ dạng text khó đọc sang CSV có cấu trúc, có thể query được bằng Athena, với chi phí thấp và không cần đầu tư hạ tầng phức tạp.

**Nguồn tham khảo:** https://aws.amazon.com/blogs/database/automate-postgresql-audit-log-extraction-and-analysis-with-amazon-s3/
