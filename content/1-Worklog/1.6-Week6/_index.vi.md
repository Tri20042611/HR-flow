---
title: "Worklog Tuần 6"
date: 2026-05-25
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

### Mục tiêu tuần 6:

- Đào sâu về **IAM**: user, group, role, policy (managed vs inline), Access Analyzer và IAM Access Advisor.
- Áp dụng **MFA** cho tất cả người dùng có quyền nhạy cảm và cấu hình chính sách mật khẩu mạnh.
- Tìm hiểu **AWS IAM Identity Center (SSO)**, liên kết với Microsoft AD và gán permission set cho nhiều account.

> Tuần này mình có 4 buổi tập trung vào Identity & Access Management - chủ đề mình thấy khô khan nhưng cực kỳ quan trọng với người làm security.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 3 | - Ôn lại cấu trúc IAM: principal, action, resource, condition <br> - Phân biệt **AWS managed policy**, **Customer managed policy** và **Inline policy** <br> - Tạo group `Developers`, `SecurityAuditors`, `ReadOnly`, viết policy riêng cho từng group theo nguyên tắc **least privilege** | 26/05/2026 | 26/05/2026 | <https://000002.awsstudygroup.com/> |
| 4 | - Bật **MFA** cho root user bằng thiết bị phần cứng giả lập trên điện thoại <br> - Bắt buộc MFA cho group `Developers` qua IAM policy với điều kiện `aws:MultiFactorAuthPresent` <br> - Cấu hình **password policy**: độ dài tối thiểu 14, có chữ hoa/thường/số/ký tự đặc biệt, hết hạn 90 ngày <br> - Thử đăng nhập không có MFA để xác nhận policy chặn đúng | 27/05/2026 | 27/05/2026 | <https://000002.awsstudygroup.com/> |
| 5 | - Tìm hiểu **IAM Roles** cho cross-account access và dịch vụ AWS (EC2, Lambda, Glue...) <br> - Tạo role `EC2-S3-ReadOnly` cho phép EC2 assume để đọc S3 mà không cần access key <br> - Dùng **IAM Access Analyzer** để phát hiện resource bị share với principal ngoài tổ chức <br> - Xem **IAM Access Advisor** của từng user để biết service nào đang được dùng, từ đó siết lại policy | 28/05/2026 | 28/05/2026 | <https://000002.awsstudygroup.com/> |
| 6 | - Tổng quan **AWS IAM Identity Center (SSO)**: tại sao nên dùng thay cho IAM user khi có nhiều account <br> - Enable IAM Identity Center, tạo user `security-lead@example.com` <br> - Tạo **permission set** `DeveloperReadOnly` và `SecurityAdmin` <br> - Gán permission set cho user trên nhiều account, đăng nhập bằng SSO portal để chuyển account mượt mà | 29/05/2026 | 29/05/2026 | <https://000002.awsstudygroup.com/> |

### Kết quả đạt được tuần 6:

- Nắm chắc cấu trúc JSON của IAM policy và viết được policy theo nguyên tắc **least privilege** cho từng nhóm.
- Bật MFA cho root user và bắt buộc MFA cho group Developers bằng `aws:MultiFactorAuthPresent`.
- Cấu hình **password policy** mạnh cho account (độ dài, ký tự đặc biệt, thời hạn).
- Tạo và assume thành công **IAM Role** cho EC2 thay cho access key cứng.
- Dùng được **IAM Access Analyzer** để rà soát resource public và phát hiện sai sót chia sẻ ngoài ý muốn.
- Triển khai **IAM Identity Center (SSO)** cơ bản, tạo permission set, gán cho nhiều account.

### Khó khăn và bài học:

- Lúc đầu viết policy gán cho group có dùng `Resource: "*"` quá rộng, mentor nhắc mới siết lại còn các resource cụ thể. Đây là điểm dễ mắc khi mới làm quen IAM.
- IAM Role cho EC2 phải được EC2 assume bằng metadata service, mình cố nhúng access key vào user-data thì mentor chặn lại và giải thích vì sao phải dùng role.
- IAM Identity Center từng gọi là AWS Single Sign-On, có nhiều tài liệu cũ nên đọc phải đối chiếu kỹ với console hiện tại.