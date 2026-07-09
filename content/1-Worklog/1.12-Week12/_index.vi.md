---
title: "Worklog Tuần 12"
date: 2026-07-06
weight: 12
chapter: false
pre: " <b> 1.12. </b> "
---

### Mục tiêu tuần 12:

- Hoàn thiện **frontend SPA** (candidate app + HR dashboard) host trên S3, có thể demo end-to-end trên cloudflared tunnel.
- Chạy **demo toàn bộ hệ thống HireFlow** từ upload CV đến HR xem ranking, ghi lại video/screenshots làm báo cáo.
- Viết **báo cáo worklog cuối khóa** tổng kết 12 tuần thực tập và đồ án HireFlow.

> Tuần cuối khóa, 4 buổi. 2 buổi đầu focus sản phẩm (frontend + demo), 2 buổi cuối viết báo cáo worklog tổng kết.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Dựng **Candidate SPA** (`candidate-app/`): form upload CV, gọi API `GET /apply/presigned-url` lấy URL, upload trực tiếp lên S3 Quarantine <br> - Hiển thị trạng thái apply realtime (poll API `GET /candidates/{id}`) <br> - Dựng **HR dashboard SPA**: bảng danh sách candidate, filter theo `job_id`, sort theo `score`, xem chi tiết CV <br> - Host cả 2 SPA lên **S3 static website**: `apply-public-read`, set `index.html` + CORS | 06/07/2026 | 06/07/2026 | <https://docs.aws.amazon.com/AmazonS3/latest/userguide/WebsiteHosting.html> |
| 3 | - Demo end-to-end HireFlow: tạo 1 JD mẫu qua API, upload 3–5 CV test từ `cv-test/` <br> - Quan sát pipeline trên CloudWatch Logs, xác nhận score + email đến đúng ứng viên <br> - Dùng **cloudflared tunnel** trỏ domain `hireflow-dev` vào S3 static, demo cho mentor xem <br> - Ghi lại **screenshots** các bước: upload form, HR dashboard ranking, email inbox, CloudWatch logs | 07/07/2026 | 07/07/2026 | <https://docs.aws.amazon.com/AmazonCloudWatch/> |
| 4 | - Viết **báo cáo worklog cuối khóa**: tổng kết 12 tuần, kiến trúc HireFlow, kết quả đạt được, lessons learned, hướng phát triển <br> - Tổng hợp worklog từ tuần 1–12 vào file PDF, có hình ảnh minh họa <br> - Lên **slide thuyết trình** (10–12 slide) cho buổi báo cáo cuối khóa <br> - Review lại README.md của repo HireFlow, đảm bảo onboarding được cho người mới | 08/07/2026 | 08/07/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 6 | - **Buổi báo cáo cuối khóa** với mentor + các bạn cùng khóa <br> - Trình bày kiến trúc HireFlow, demo trực tiếp trên cloudflared tunnel <br> - Nhận feedback từ mentor: về trade-off CORS/Auth, LLM endpoint third-party, scope của Cognito <br> - Cleanup tài nguyên AWS account sandbox (xoá stack `hireflow-dev` và các resource test) <br> - Ghi nhận các hướng cải thiện cho giai đoạn sau: re-enable GuardDuty, structured resume parsing, batch scoring, multi-channel ingest | 10/07/2026 | 10/07/2026 | <https://cloudjourney.awsstudygroup.com/> |

### Kết quả đạt được tuần 12:

- Hoàn thiện **2 SPA frontend** (candidate + HR dashboard) host trên S3 static website, có thể truy cập qua cloudflared tunnel.
- Demo thành công **end-to-end HireFlow**: upload 3–5 CV → validate → OCR → LLM score → DynamoDB ranking → email xác nhận → HR xem bảng candidate.
- Viết và nộp **báo cáo worklog cuối khóa** tổng kết 12 tuần, có hình ảnh minh họa từng giai đoạn.
- Bảo vệ đồ án trước mentor, nhận feedback tích cực về kiến trúc decouple bằng SQS và hạ tầng IaC bằng SAM.
- Hoàn thành **khóa thực tập First Cloud AI Journey**, có sản phẩm demo + worklog đầy đủ để tham chiếu sau này.

### Khó khăn và bài học:

- CloudFront chưa bật vì cost, dùng **cloudflared tunnel** cho demo dev — ghi nhận là trade-off chấp nhận được vì CV traffic thấp và ứng dụng nội bộ.
- Lúc demo email SES, một số ứng viên test bị vào **spam folder** vì SPF/DKIM chưa cấu hình chuẩn — đã ghi nhận vào "Known limitations", production cần setup custom domain + DKIM.
- Phần báo cáo worklog cuối khóa tốn nhiều thời gian tổng hợp vì 12 tuần có nhiều thông tin — bài học là viết worklog ngắn gọn từng tuần để cuối khóa chỉ cần compile, không phải viết lại từ đầu.
- Cleanup cuối khóa dễ quên: stack SAM xoá xong nhưng Secrets Manager secret vẫn còn, S3 bucket có versioning vẫn giữ object — phải dùng script `scripts/cleanup.sh` để xoá hết, kể cả các versioned object.