---
title: "Bản đề xuất — Dự án HireFlow AI"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

### 1. Tóm tắt điều hành

HireFlow AI được thiết kế nhằm tự động hóa quy trình sàng lọc hồ sơ ứng viên (CV) cho bộ phận tuyển dụng, thay thế các thao tác thủ công lặp lại bằng nền tảng AWS Serverless tích hợp trí tuệ nhân tạo. Hệ thống hỗ trợ tiếp nhận CV từ nhiều kênh (web form, email, tải lên trực tiếp), tự động trích xuất văn bản qua Amazon Textract, chấm điểm ứng viên nhờ mô hình ngôn ngữ lớn (LLM), và trình bày danh sách xếp hạng trên bảng điều khiển dành cho nhà tuyển dụng (HR Dashboard). Nền tảng phục vụ tối đa 10–15 tin tuyển dụng đang mở và xử lý đồng thời lên tới 200 CV/ngày, với khả năng mở rộng theo nhu cầu thực tế.

### 2. Tuyên bố vấn đề

**Vấn đề hiện tại**

Quy trình tuyển dụng hiện tại phụ thuộc vào các thao tác thủ công: nhận CV qua email, tải xuống và đọc từng tệp, sao chép thông tin sang bảng tính, đối chiếu với yêu cầu công việc (JD), rồi chấm điểm thủ công. Trung bình mỗi CV mất 8–12 phút xử lý; khi có 100 hồ sơ, một nhà tuyển dụng cần khoảng 15–20 giờ làm việc liên tục chỉ để sàng lọc ban đầu. Việc đánh giá thiếu nhất quán giữa các ứng viên, dễ bỏ sót hồ sơ tiềm năng và không có dấu vết kiểm toán (audit trail).

**Giải pháp**

Nền tảng sử dụng Amazon S3 làm kho lưu trữ trung gian (quarantine zone) để tiếp nhận CV, AWS Lambda kết hợp Amazon Textract để trích xuất văn bản từ PDF và DOCX, Amazon SQS để tách rời các giai đoạn xử lý, AWS Lambda tích hợp mô hình ngôn ngữ lớn (LLM) để chấm điểm ứng viên dựa trên JD, và Amazon DynamoDB để lưu trữ dữ liệu có cấu trúc. Bảng điều khiển React (HR Dashboard) được lưu trữ trên Amazon S3 phân phối qua CloudFront, bảo mật bằng Amazon Cognito. Đường ống xử lý hàng loạt được tự động hóa hoàn toàn, kết quả trả về trong vòng 2–3 phút/CV.

**Lợi ích và hoàn vốn đầu tư (ROI)**

Giải pháp cắt giảm 80% thời gian sàng lọc ban đầu (từ 10 phút xuống dưới 2 phút mỗi CV), đảm bảo tính nhất quán trong đánh giá nhờ tiêu chí chấm điểm thống nhất qua LLM, đồng thời tạo lịch sử đầy đủ phục vụ kiểm toán và phân tích về sau. Chi phí hạ tầng ước tính dưới 5 USD/tháng cho khối lượng 200 CV/ngày (theo AWS Pricing Calculator), tổng cộng dưới 60 USD/năm. Thời gian hoàn vốn ước tính dưới 6 tháng dựa trên hiệu suất nhân sự tiết kiệm được.

### 3. Kiến trúc giải pháp

**Tổng quan**

HireFlow AI áp dụng kiến trúc AWS Serverless theo mô hình hàng đợi (queue-based pipeline) để đảm bảo khả năng mở rộng, chịu lỗi và tách rời giữa các giai đoạn. CV được ứng viên tải lên qua presigned URL thẳng vào S3 Quarantine bucket (vùng cách ly), tránh đi qua API Gateway để giảm tải và vượt giới hạn payload 6 MB. Sau khi vượt qua lớp xác thực định dạng, CV được chuyển sang S3 Clean bucket và nạp vào hàng đợi xử lý trích xuất.

**Sơ đồ kiến trúc**

![HireFlow AI Architecture](/images/2-Proposal/hireflow_architecture.png)

**Dịch vụ AWS sử dụng**

- **Amazon S3 (2 bucket):** Quarantine zone tiếp nhận file thô; Clean zone chứa file đã xác thực cho pipeline xử lý.
- **AWS Lambda (7 hàm):** File-validator (bảo mật), Extract (Textract), Extract-complete (gộp kết quả), Score (LLM), Save-and-notify (ghi DB + email), Candidates (CRUD cho HR Dashboard), JD-management (quản lý tin tuyển dụng), Get-presigned-url (upload URL).
- **Amazon API Gateway:** HTTP API làm lớp giao tiếp giữa React SPA và Lambda, có Cognito authorizer.
- **Amazon SQS (2 hàng đợi):** Score-queue đệm trước giai đoạn chấm điểm; Save-queue đệm trước giai đoạn lưu và thông báo.
- **Amazon Textract:** Trích xuất văn bản và cấu trúc bảng từ CV PDF/DOCX.
- **Amazon DynamoDB (2 bảng):** Candidates lưu hồ sơ đã chấm; Jobs lưu tin tuyển dụng + JD + tiêu chí chấm.
- **Amazon Cognito:** User pool cho HR (5 tài khoản), user pool riêng cho ứng viên (tùy chọn).
- **Amazon SNS:** Textract notification topic, HR alert topic gửi email khi pipeline lỗi.
- **Amazon CloudFront + S3 Hosting:** Phân phối HR Dashboard (React SPA).
- **AWS Secrets Manager:** Lưu API key cho LLM provider.
- **AWS KMS:** Mã hóa dữ liệu trong S3, DynamoDB, SQS.

**Thiết kế thành phần**

- **Lớp tiếp nhận (Zone 1):** Ứng viên yêu cầu presigned URL từ Lambda `get-presigned-url`, sau đó tải file trực tiếp lên S3 Quarantine qua HTTPS PUT.
- **Lớp bảo mật (Zone 2):** EventBridge rule bắt sự kiện S3 ObjectCreated ở prefix `web/`; Lambda `file-validator` kiểm tra MIME (magic bytes), kích thước ≤ 5 MB, dedup SHA-256, sau đó copy sang Clean và xóa ở Quarantine.
- **Lớp xử lý (Zone 3):** Lambda `extract` đọc file ở Clean, gọi Textract async, lưu JobId; SNS Textract gọi Lambda `extract-complete` gộp kết quả, gửi SQS message sang hàng đợi chấm điểm.
- **Lớp AI:** Lambda `score` nhận SQS message, đọc JD từ DynamoDB, gọi LLM (OpenAI-compatible endpoint), lưu điểm và lý do vào DynamoDB, gửi SQS message sang hàng đợi lưu.
- **Lớp lưu trữ & thông báo:** Lambda `save-and-notify` ghi bản ghi cuối cùng vào DynamoDB và gửi email xác nhận cho ứng viên qua SES.
- **Lớp giao diện:** HR Dashboard React SPA gọi API Gateway → Lambda `candidates`/`jd-management` đọc/ghi DynamoDB, được bảo vệ bởi Cognito.

### 4. Triển khai kỹ thuật

**Các giai đoạn triển khai**

Dự án gồm 4 giai đoạn chính, mỗi giai đoạn có sản phẩm bàn giao rõ ràng:

1. **Nghiên cứu và thiết kế kiến trúc (Tuần 1–2):** Khảo sát quy trình tuyển dụng hiện tại, đánh giá các dịch vụ AWS phù hợp, lập sơ đồ kiến trúc tổng thể và sơ đồ tuần tự cho từng flow chính.
2. **Tính toán chi phí và đánh giá tính khả thi (Tuần 3):** Dùng AWS Pricing Calculator ước tính chi phí cho các kịch bản tải thấp/trung bình/cao; xác định điểm tắc nghẽn tiềm ẩn và cách giảm chi phí (Lambda memory sizing, S3 lifecycle, SQS batching).
3. **Điều chỉnh kiến trúc để tối ưu (Tuần 4–5):** Tinh chỉnh thiết kế — giảm tải cho Lambda bằng cách tách đường ống thành nhiều stage với SQS, tái cấu trúc IAM role theo nguyên tắc least-privilege, bổ sung cơ chế retry và dead-letter queue.
4. **Phát triển, kiểm thử và triển khai (Tuần 6–12):** Lập trình Lambda bằng Python 3.12 với boto3, sử dụng AWS SAM làm IaC để định nghĩa toàn bộ hạ tầng; kiểm thử từng Lambda với event mẫu; chạy end-to-end trên tài khoản thật với 10 CV mẫu.

**Yêu cầu kỹ thuật**

- **AWS SAM:** Định nghĩa toàn bộ stack (Lambda, S3, DynamoDB, SQS, SNS, Cognito, API Gateway, CloudFront) bằng `template.yaml`.
- **AWS Lambda:** Python 3.12, boto3 SDK, structured logging, X-Ray tracing.
- **Amazon Textract:** `StartDocumentAnalysis` với feature `TABLES` + `FORMS`, polling hoặc SNS callback cho async completion.
- **LLM Integration:** OpenAI-compatible endpoint (đã provision sẵn qua 9router.xjanua.me), lưu key trong Secrets Manager, prompt engineering cho chấm điểm 4 tiêu chí (skills, experience, education, fit).
- **Frontend:** React 19 + Vite + Amazon Cognito Identity SDK, build artifact deploy lên S3 phân phối qua CloudFront.
- **Bảo mật:** IAM least-privilege, S3 block public access, S3 + DynamoDB + SQS encryption với KMS, VPC endpoint không bắt buộc nhưng network ACL được xem xét.
- **Observability:** CloudWatch Logs (giữ 7 ngày), CloudWatch Metrics (Lambda duration + error count), CloudWatch Alarm khi Lambda error rate > 5%.

### 5. Lộ trình & Mốc triển khai

Dự án chạy xuyên suốt 12 tuần, align với lịch thực tập.

**Tuần 1–8: Giai đoạn học tập**

- **Tuần 1:** Làm quen với AWS — EC2, IAM, CLI, Budget alerts
- **Tuần 2:** Deep dive EC2 & EBS, Session Manager, Linux cơ bản
- **Tuần 3:** Amazon S3 — Versioning, Lifecycle, Replication, Block Public Access, pre-signed URL, SSE-KMS
- **Tuần 4:** Amazon VPC — CIDR, public/private subnet, IGW, NAT, SG vs NACL, VPC Endpoint
- **Tuần 5:** RDS Multi-AZ + Read Replica và DynamoDB partition/sort key, GSI, Streams
- **Tuần 6:** IAM chuyên sâu — least privilege, MFA, IAM Role, Access Analyzer, Identity Center SSO
- **Tuần 7:** KMS, GuardDuty, Security Hub, WAF, CloudTrail, CloudWatch
- **Tuần 8:** Serverless với Lambda + API Gateway + DynamoDB và CloudFormation IaC

**Tuần 9–12: Giai đoạn phát triển dự án**

| Tuần | Mốc | Sản phẩm chính |
|------|------|----------------|
| **Tuần 9** | Khởi động & Kiến trúc | - Làm quen Kiro IDE (AI code generation) <br> - Học Amazon SQS (standard queue, visibility timeout, DLQ) <br> - Brainstorm và chốt kiến trúc HireFlow AI <br> - Ánh xạ mô hình SQS vào pipeline extract→score→save |
| **Tuần 10** | Cài đặt Backend với SAM | - Khởi tạo project HireFlow bằng SAM CLI (`hireflow-sam/`) <br> - Deploy SAM stack: 6 S3 bucket, 2 DynamoDB table, 2 SQS + 2 DLQ <br> - Viết Lambda `file-validator` (kiểm tra MIME, SHA-256 dedup) <br> - Tích hợp Amazon Textract để OCR CV <br> - Pipeline: Upload → Validate → Extract |
| **Tuần 11** | Hoàn thiện Pipeline | - Hoàn thành pipeline async OCR → score → save <br> - Tích hợp LLM scoring qua OpenAI-compatible endpoint <br> - Cấu hình Amazon SES gửi email xác nhận ứng viên <br> - Setup DLQ cho cả 2 SQS queue với retry logic <br> - Luồng trạng thái hoàn chỉnh: uploaded → extracting → extracted → scoring → scored → saved |
| **Tuần 12** | Frontend & Demo | - Build Candidate SPA + HR Dashboard trên S3 static website <br> - Demo end-to-end: 3–5 CV → validate → OCR → LLM score → DynamoDB → email <br> - Trình bày kiến trúc HireFlow qua cloudflared tunnel <br> - Nộp báo cáo worklog cuối kỳ |

### 6. Ước tính ngân sách

**Chi phí hạ tầng (ước tính cho 200 CV/ngày)**

| Danh mục | Dịch vụ | Chi phí ước tính |
|---|---|---|
| Lưu trữ & CDN | S3 (2 bucket) | ~$2.50/tháng |
| Lưu trữ & CDN | CloudFront + S3 Hosting | ~$1.00/tháng |
| Tính toán | Lambda (7 hàm) | ~$5.00/tháng |
| AI/ML | Textract | ~$15.00/tháng |
| Mạng | API Gateway (HTTP API) | ~$3.50/tháng |
| Mạng | SQS (2 hàng đợi) | ~$0.00/tháng |
| Mạng | SNS (2 topic) | ~$0.00/tháng |
| Cơ sở dữ liệu | DynamoDB (2 bảng) | ~$8.00/tháng |
| Bảo mật | Secrets Manager | ~$0.40/tháng |
| Bảo mật | KMS | ~$1.00/tháng |
| Giám sát | CloudWatch | ~$2.00/tháng |
| **Tổng** | | **~$38.40/tháng** |

**Lưu ý quan trọng:**

1. Textract là chi phí chiếm chủ yếu ở $0.015/trang — tăng tuyến tính với số CV xử lý.
2. SQS, SNS và Cognito đều nằm trong free tier ở khối lượng dự kiến.
3. Lambda có 1 triệu request + 400K GB-giây miễn phí mỗi tháng.
4. Chi phí tăng chủ yếu ở Textract và Lambda khi lượng CV tăng.

**Phần cứng:** Không có — toàn bộ xử lý trên cloud, ứng viên dùng trình duyệt web.

### 7. Đánh giá rủi ro

**Ma trận rủi ro**

| Rủi ro | Ảnh hưởng | Xác suất |
|---|---|---|
| Textract quota vượt free-tier | Trung bình | Thấp |
| LLM API timeout/down | Cao | Thấp |
| Vượt ngân sách do Textract cost | Trung bình | Trung bình |
| CV giả mạo / chèn malware | Cao | Thấp |
| Race condition khi nhiều CV cùng lúc | Trung bình | Thấp |

**Chiến lược giảm thiểu**

- **Quota:** Bật CloudWatch alarm cho Textract `ThrottledCount`, xin quota raise khi cần.
- **LLM availability:** Triển khai circuit breaker trong Lambda `score`; hàng đợi SQS giữ message retry tối đa 3 lần trước khi vào DLQ.
- **Secret hygiene:** Truy cập key qua biến môi trường từ Secrets Manager, không log; xoay key mỗi 90 ngày.
- **Chi phí:** Cảnh báo ngân sách AWS Billing ở mức $5/tháng; tự động tắt Textract nếu vượt.
- **Malware:** Pipeline xác thực kiểm tra MIME bằng magic bytes (không tin Content-Type header), lưu CV ở Quarantine 7 ngày trước khi xóa để HR review khi cần.
- **Concurrency:** DynamoDB conditional write trên SHA-256 hash để chống duplicate; idempotency key trên mỗi message.

**Kế hoạch dự phòng**

- Quay lại quy trình thủ công nếu hệ thống sự cố hơn 24 giờ.
- Toàn bộ hạ tầng được IaC qua SAM, khôi phục trong dưới 30 phút bằng `sam deploy`.
- Lưu CV đã xử lý ở Clean bucket 90 ngày làm backup.

### 8. Kết quả kỳ vọng

- **Cải tiến kỹ thuật:** Giảm 80% thời gian sàng lọc CV; chấm điểm nhất quán qua LLM; có audit trail đầy đủ; mở rộng được lên 500 CV/ngày mà không tăng chi phí đáng kể.
- **Giá trị dài hạn:** Dữ liệu ứng viên lịch sử trong DynamoDB có thể dùng để phân tích xu hướng tuyển dụng, đào tạo mô hình scoring riêng; nền tảng có thể tái sử dụng cho các bài toán tương tự (sàng lọc đơn xét duyệt, đánh giá nhà cung cấp).
