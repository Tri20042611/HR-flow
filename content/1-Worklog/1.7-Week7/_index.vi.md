---
title: "Worklog Tuần 7"
date: 2026-06-01
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---

### Mục tiêu tuần 7:

- Nắm vững **AWS KMS**: customer-managed key, key policy, grant, alias và rotation.
- Triển khai các dịch vụ phát hiện & bảo vệ: **GuardDuty**, **Security Hub**, **WAF** và **Shield Standard**.
- Làm quen giám sát hệ thống với **CloudTrail**, **CloudWatch Logs/Metrics/Alarms** và **EventBridge**.

> 5 buổi liên tiếp, đây là tuần mình thấy "đã làm security" nhất từ đầu khóa vì gần như mọi dịch vụ đều liên quan trực tiếp đến bảo vệ và giám sát.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Ôn tổng quan mô hình bảo mật AWS: **shared responsibility model** <br> - Tìm hiểu **AWS KMS**: symmetric key, asymmetric key, alias, key policy <br> - Tạo **customer-managed CMK**, alias `fcaj-key`, bật **key rotation hằng năm** <br> - Dùng CMK để mã hóa EBS volume của EC2 mới và xác nhận trong console | 01/06/2026 | 01/06/2026 | <https://000033.awsstudygroup.com/> |
| 3 | - Dùng CMK mã hóa thử S3 bucket, RDS snapshot <br> - Thực hành **grant** quyền dùng key cho một IAM role qua CloudFormation <br> - Thử revoke grant và xác nhận EC2 không giải mã được EBS nữa | 02/06/2026 | 02/06/2026 | <https://000033.awsstudygroup.com/> |
| 4 | - Bật **AWS CloudTrail** ở cả region với multi-region trail, lưu log vào S3 riêng <br> - Bật **CloudTrail Insights** và **CloudTrail Lake** <br> - Tạo **CloudWatch Logs group** cho CloudTrail, viết metric filter đếm số lần `ConsoleLogin` <br> - Tạo **CloudWatch Alarm** gửi SNS notification khi có `Root` login | 03/06/2026 | 03/06/2026 | <https://000008.awsstudygroup.com/> |
| 5 | - Bật **GuardDuty** ở region `ap-southeast-1`, bật thêm **S3 Protection** và **RDS Protection** <br> - Tạo tình huống test: chạy SSH brute-force từ một IP đáng ngờ, đợi GuardDuty phát hiện `SSH brute force` finding <br> - Bật **Security Hub** ở region, liên kết GuardDuty, IAM Access Analyzer, Inspector thành một bảng điều khiển chung <br> - Review các finding, phân loại critical/high/medium/low | 04/06/2026 | 04/06/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 6 | - Triển khai 1 ALB phía trước EC2 web tier <br> - Bật **AWS WAF** gắn vào ALB, thêm rule chặn request có chứa `User-Agent: BadBot` <br> - Tạo **rate-based rule** chặn IP nào gửi quá 2000 request/5 phút <br> - Bật **Shield Standard** (miễn phí) cho ALB <br> - Tổng kết tuần: dọn dẹp resource, xóa rule test, tắt GuardDuty để tránh phát sinh phí | 05/06/2026 | 05/06/2026 | <https://cloudjourney.awsstudygroup.com/> |

### Kết quả đạt được tuần 7:

- Nắm vững **shared responsibility model** của AWS, hiểu rõ phần nào AWS chịu, phần nào khách hàng chịu.
- Tạo và quản lý **KMS customer-managed key** với key policy, alias, rotation.
- Dùng KMS key để mã hóa EBS, S3 bucket, RDS snapshot.
- Triển khai CloudTrail multi-region, CloudWatch Logs, Metrics, Alarm kết hợp SNS.
- Bật **GuardDuty** thành công, mô phỏng được tình huống SSH brute-force và đọc finding.
- Bật **Security Hub**, hợp nhất finding từ nhiều dịch vụ security.
- Triển khai **WAF** trước ALB với rule chặn bot và rate-based rule.

### Khó khăn và bài học:

- Lúc đầu gắn WAF trực tiếp vào EC2 thì không được, phải có **Application Load Balancer** hoặc **CloudFront** làm lớp trước. Bài học về cách AWS thiết kế từng lớp bảo vệ.
- CloudTrail Insights chỉ phát hiện hành vi bất thường dựa trên baseline, phải chờ vài ngày để có dữ liệu baseline đủ tốt.
- GuardDuty mất khoảng 5-15 phút để phát hiện SSH brute-force, nên không thể "test" theo kiểu real-time, cần kiên nhẫn đợi.