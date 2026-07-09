---
title: "Worklog Tuần 11"
date: 2026-06-29
weight: 11
chapter: false
pre: " <b> 1.11. </b> "
---

### Mục tiêu tuần 11:

- Hoàn thiện **pipeline OCR → score → save** của HireFlow, đi từ raw CV PDF đến candidate row có điểm trong DynamoDB.
- Tích hợp **LLM scoring** (OpenAI-compatible API) để chấm điểm CV theo Job Description.
- Gửi **email xác nhận** cho ứng viên qua SES khi CV đủ điểm được lưu thành công.

> Tuần này có 4 buổi, bỏ Thứ 5. Mỗi buổi tương ứng một tầng trong pipeline async, từ OCR callback cho đến email cuối cùng.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Viết Lambda **extract-complete** nhận SNS callback từ Textract (`JobStatus=SUCCEEDED`) <br> - Gọi `GetDocumentTextDetection` để lấy text đầy đủ từ Textract job <br> - Parse text → lưu vào S3 `OCR Results` và push message `{candidate_id, job_id, cv_text}` vào **SQS `score-queue`** <br> - Cập nhật DynamoDB row: status `extracting → extracted` | 29/06/2026 | 29/06/2026 | <https://docs.aws.amazon.com/textract/> |
| 3 | - Viết Lambda **score** consumer từ `score-queue`, dùng **Amazon Bedrock (Claude)** hoặc endpoint OpenAI-compatible (`9router.xjanua.me/v1`, model `Tri_Bui`) <br> - Prompt gồm `JD + cv_text` → LLM trả JSON `{score, reasoning, strengths, gaps}` <br> - Validate JSON output (chống LLM hallucinate), push kết quả sang **SQS `save-queue`** nếu `score > threshold` (mặc định 60) <br> - Status trong DDB: `extracted → scoring → scored` (hoặc `low_score` nếu dưới ngưỡng) | 30/06/2026 | 30/06/2026 | <https://docs.aws.amazon.com/bedrock/> |
| 4 | - Viết Lambda **save-notify** consumer từ `save-queue`, update row DDB cuối cùng với `score, reasoning, strengths, gaps`, status `scored → saved` <br> - Cấu hình **Amazon SES**: verify domain/sender, request production access (sandbox trước) <br> - Gửi email template cho ứng viên với nội dung: "Cảm ơn bạn đã apply, CV của bạn đã được AI sơ duyệt..." <br> - Test với 1 CV thật, kiểm tra email đến inbox (và spam folder) | 01/07/2026 | 01/07/2026 | <https://docs.aws.amazon.com/ses/> |
| 6 | - Cấu hình **DLQ** cho cả `score-queue` và `save-queue`, set `maxReceiveCount = 3` <br> - Cố tình inject message lỗi (payload thiếu field `cv_text`, hoặc LLM trả JSON không parse được) <br> - Quan sát message được retry 3 lần rồi chuyển sang DLQ <br> - Viết Lambda **dlq-handler** (optional) log lỗi chi tiết, alert qua SNS cho HR biết pipeline đang fail <br> - Cập nhật `README.md` với sơ đồ kiến trúc ASCII và troubleshooting guide | 03/07/2026 | 03/07/2026 | <https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/> |

### Kết quả đạt được tuần 11:

- Hoàn thiện **pipeline async OCR → score → save**: CV PDF đi từ S3 Quarantine đến DynamoDB row có đầy đủ `score, reasoning, strengths, gaps`.
- Tích hợp thành công **LLM scoring** qua endpoint OpenAI-compatible, có validate output chống parse error.
- **Amazon SES** gửi email xác nhận thành công cho ứng viên ngay sau khi CV đủ điểm được lưu.
- Cấu hình **DLQ** cho cả 2 SQS, test được cả flow retry → DLQ khi message lỗi.
- Mỗi candidate có **UUID candidate_id** với status flow đầy đủ: `uploaded → extracting → extracted → scoring → scored → saved` (hoặc `low_score`, `extraction_failed`).

### Khó khăn và bài học:

- Textract SNS callback trong SAM template không tự bind được vào Lambda, phải tạo SNS topic + subscription thủ công bằng `aws sns subscribe` sau khi deploy — đã ghi vào `scripts/setup_sns.sh` để chạy lại nhanh.
- LLM đôi khi trả text có ```json``` markdown wrapper làm parse fail; phải thêm regex strip trước khi `json.loads`. Cũng có lúc model trả thiếu field `gaps` — bài học là luôn default fallback cho field optional.
- SES ở sandbox chỉ gửi được tới email đã verify, lúc đầu test với Gmail cá nhân không nhận được vì chưa verify; phải verify từng recipient hoặc request production access.
- Việc LLM là third-party endpoint (`9router.xjanua.me`) có rủi ro downtime — đã ghi nhận vào "Known limitations" của README, production nên tự host hoặc dùng Bedrock.