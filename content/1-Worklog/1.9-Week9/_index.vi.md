---
title: "Worklog Tuần 9"
date: 2026-06-15
weight: 9
chapter: false
pre: " <b> 1.9. </b> "
---

### Mục tiêu tuần 9:

- Làm quen với **Kiro** (AI IDE hỗ trợ sinh code, refactor, viết test) để tăng tốc workflow cá nhân.
- Tìm hiểu **Amazon SQS**: standard queue, visibility timeout, long polling, Dead-Letter Queue.
- Lên ý tưởng kiến trúc cho **đồ án cuối khóa**: HireFlow AI — hệ thống tuyển dụng serverless hỗ trợ HR duyệt CV bằng AI.

> Tuần này có 4 buổi: 1 buổi làm quen công cụ (Kiro), 1 buổi học SQS, 1 buổi lên ý tưởng kiến trúc HireFlow, buổi cuối chốt phương án và phân công việc cho 3 tuần tới.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Cài đặt và cấu hình **Kiro IDE** trên máy local <br> - Tìm hiểu các chế độ: chat, agent, inline edit, spec-driven <br> - Viết một Lambda function đơn giản bằng prompt trên Kiro, so sánh output với code tự viết tay <br> - Dùng Kiro refactor một đoạn code Python cũ của tuần 8 (Lambda + DynamoDB) để clean hơn | 15/06/2026 | 15/06/2026 | <https://kiro.dev/> |
| 3 | - Đề xuất 2-3 ý tưởng đồ án cuối khóa: <br>&emsp; + **HireFlow AI** — serverless recruitment platform, HR duyệt CV nhờ AI (Textract OCR + LLM score + SES email) <br>&emsp; + Hệ thống xử lý đơn hàng async với SQS + Lambda + DynamoDB <br>&emsp; + Pipeline ingestion log từ CloudWatch → SQS → Lambda → S3/Redshift <br> - Phân tích ưu/nhược điểm từng phương án (cost, complexity, learning value) <br> - Chốt phương án **HireFlow AI**, vẽ sơ đồ kiến trúc high-level (upload CV → validate → OCR → score → email) trên Excalidraw <br> - Phân công việc cho 3 tuần còn lại của đồ án | 16/06/2026 | 16/06/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 5 | - Tổng quan **Amazon SQS**: standard queue vs FIFO queue, message size, retention <br> - Tạo SQS standard queue `orders-queue` qua console (dùng cho việc học, sẽ đổi tên sang queue pipeline CV sau) <br> - Viết Lambda **producer** đẩy message JSON vào queue, Lambda **consumer** đọc và ghi vào DynamoDB <br> - Cấu hình **visibility timeout**, **message retention**, **long polling (20s)** <br> - Test với batch lớn (~500 message) để quan sát hành vi consumer | 18/06/2026 | 18/06/2026 | <https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/> |
| 6 | - Tạo **Dead-Letter Queue** `orders-dlq` cho `orders-queue`, cấu hình `maxReceiveCount = 3` <br> - Cố tình đẩy message lỗi để quan sát flow: main queue → DLQ → Lambda xử lý riêng <br> - Map pattern SQS vừa học vào kiến trúc HireFlow: 2 queue `score-queue` + `save-queue` nằm giữa `extract → score → save-notify` <br> - Hoàn thiện file `README.md` của đồ án HireFlow: mục tiêu, kiến trúc, công nghệ (8 Lambda, 6 S3, 2 DDB, SQS, Textract, SES), kế hoạch 3 tuần <br> - Tổng kết tuần, chuẩn bị cho Tuần 10 (chuyển sang dùng SAM) | 19/06/2026 | 19/06/2026 | <https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/> |

### Kết quả đạt được tuần 9:

- Cài đặt và sử dụng thành thạo **Kiro IDE** ở các chế độ chat, agent, inline edit; tiết kiệm được ~30% thời gian viết code so với tự viết tay.
- Dùng Kiro refactor code Lambda tuần 8, output clean hơn, có docstring đầy đủ, đã verify lại bằng test thủ công.
- Chốt được ý tưởng **đồ án cuối khóa: HireFlow AI — Serverless Recruitment Platform** với sơ đồ kiến trúc high-level và phân công việc cho 3 tuần tới.
- Tạo và vận hành **SQS standard queue** với producer/consumer Lambda, hiểu rõ visibility timeout và long polling — sẵn sàng áp dụng vào pipeline extract→score→save của HireFlow.
- Cấu hình **Dead-Letter Queue** và quan sát được message lỗi được chuyển sang DLQ đúng sau 3 lần retry.

### Khó khăn và bài học:

- Kiro sinh code nhanh nhưng đôi lúc "hallucinate" — gọi API không tồn tại hoặc import thiếu. Bài học: luôn review code do AI sinh trước khi chạy thật.
- SQS consumer Lambda mặc định batch size = 10, lúc đầu quên nên throughput thấp; tăng lên batch size 10 và concurrency 5 mới xử lý kịp 500 message.
- Lúc đầu tạo DLQ xong nhưng quên set `redrive policy` trên main queue, message lỗi cứ nằm lại main queue mà không chuyển sang DLQ — phải sửa lại policy mới hoạt động.