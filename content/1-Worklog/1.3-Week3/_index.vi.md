---
title: "Worklog Tuần 3"
date: 2026-05-04
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---

### Mục tiêu tuần 3:

- Nắm vững **Amazon S3**: bucket, object, storage class, versioning, lifecycle, replication.
- Thực hành S3 theo góc nhìn an ninh mạng: Block Public Access, pre-signed URL, SSE-KMS, Bucket Policy.
- So sánh các dịch vụ storage để chọn đúng cho từng bài toán.

> Tuần này mình có 4 buổi thực hành, toàn bộ xoay quanh S3 vì đây là dịch vụ hay gặp nhất trong các bài toán security.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Tìm hiểu S3 tổng quan: bucket, object, key, prefix, quy tắc đặt tên bucket toàn cục <br> - Phân biệt các storage class: Standard, Standard-IA, One Zone-IA, Glacier Instant Retrieval, Glacier Deep Archive <br> - Thực hành: tạo bucket đầu tiên, upload object qua console & CLI | 04/05/2026 | 04/05/2026 | <https://000057.awsstudygroup.com/vi/> |
| 3 | - Bật **Versioning** trên bucket để giữ lịch sử object <br> - Cấu hình **Lifecycle rule** tự chuyển object sang Glacier sau 30 ngày và expire sau 365 ngày <br> - Cấu hình **Cross-Region Replication (CRR)** sang bucket ở region khác <br> - Thiết lập **Event notification** gửi sang SNS topic khi có object mới | 05/05/2026 | 05/05/2026 | <https://000057.awsstudygroup.com/vi/> |
| 5 | - Bật **Block Public Access** ở cả account-level lẫn bucket-level <br> - Viết **Bucket Policy** cho phép một IAM role cụ thể được đọc mà không cần pre-signed URL <br> - Phát hành **pre-signed URL** có thời hạn 15 phút để chia sẻ file nội bộ <br> - Bật **Server Access Logging** và **CloudTrail Data Events** cho S3 <br> - Thực hành: thử đăng nhập bằng IAM user không có quyền `s3:GetObject` để xác nhận policy chặn đúng | 07/05/2026 | 07/05/2026 | <https://www.youtube.com/watch?v=IIgMX2ILK6g> |
| 6 | - Bật **Default encryption với SSE-KMS** dùng customer-managed key <br> - Thử truy cập bằng IAM user không có quyền `kms:Decrypt` để xác nhận policy hoạt động <br> - Tạo **EFS**, mount từ 2 EC2 ở 2 AZ khác nhau, ghi file từ EC2 này và đọc từ EC2 kia <br> - So sánh đặc điểm **EBS / EFS / FSx / S3 / Glacier** để chốt use case phù hợp | 08/05/2026 | 08/05/2026 | <https://000033.awsstudygroup.com/> |

### Kết quả đạt được tuần 3:

- Tạo và quản lý bucket S3 thành thạo, nắm được cơ chế đặt tên toàn cục và cách chọn region hợp lý.
- Cấu hình thành công **Versioning**, **Lifecycle rule** và **Cross-Region Replication** cho S3.
- Nắm và áp dụng được các lớp bảo mật S3:
  - **Block Public Access** làm chốt chặn cuối cùng trước khi object bị public.
  - **Pre-signed URL** chia sẻ file tạm thời không cần mở public.
  - **SSE-KMS** kết hợp IAM policy kiểm soát quyền giải mã.
- Biết so sánh và lựa chọn **EBS / EFS / FSx / S3 / Glacier** theo use case thực tế.
- Có thể trình bày các lỗi bảo mật S3 phổ biến (misconfig, public bucket, thiếu log) và cách phòng tránh.

### Khó khăn và bài học:

- Cross-Region Replication yêu cầu cả 2 bucket bật Versioning, lúc đầu mình quên nên replication báo lỗi.
- Khi viết Bucket Policy nên kiểm tra cả phần `Condition` với `aws:SourceIp` hoặc `aws:PrincipalArn` để tránh vô tình cho phép rộng hơn mong muốn.
- Pre-signed URL mặc định có hiệu lực 1 giờ, phải để ý chỉnh thời gian cho phù hợp với use case.