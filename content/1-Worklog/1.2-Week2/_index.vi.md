---
title: "Worklog Tuần 2"
date: 2026-04-27
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---

### Mục tiêu tuần 2:

- Đào sâu EC2: instance family, AMI, EBS, Metadata Service và cách dùng Session Manager để truy cập an toàn không mở port SSH.
- Làm quen thao tác Linux cơ bản trên EC2 (quản lý user, quyền, process, log).

> Tuần này lịch khá mỏng, mình chỉ có 2 buổi thật sự vì giữa tuần phải dành thời gian làm bài tập ở trường. Phần còn lại mình tự đọc tài liệu ở nhà.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 3 | - EC2 đào sâu & EBS chi tiết: <br> - Phân biệt các instance family: general (t3, m5), compute optimized (c5), memory optimized (r5) <br> - Tìm hiểu IMDSv1 và IMDSv2, nhận thức rủi ro SSRF nếu để mặc định IMDSv1 <br> - Tìm hiểu EBS volume type (gp3, io2, st1, sc1) <br> - Thực hành: tạo EBS volume riêng, gắn vào EC2, format ext4 và mount vào `/data` | 28/04/2026 | 28/04/2026 | <https://000004.awsstudygroup.com/> |
| 5 | - Session Manager + Linux cơ bản: <br> - Gắn IAM role có quyền SSM cho EC2, đảm bảo SSM Agent đang chạy <br> - Đóng port 22 trên Security Group, kết nối EC2 bằng Session Manager từ console & CLI <br> - Ôn lại Linux: `useradd`, `chmod`, `chown`, `systemctl`, `journalctl`, `ss`, `df`, `du` <br> - Đọc log hệ thống tại `/var/log/` để debug một lỗi Nginx không start | 30/04/2026 | 30/04/2026 | <https://000004.awsstudygroup.com/> |

### Kết quả đạt được tuần 2:

- Phân biệt được các dòng EC2 instance và chọn được loại phù hợp cho từng bài toán.
- Hiểu sự khác biệt giữa **IMDSv1 và IMDSv2**, nhận thức rõ rủi ro SSRF nếu không tắt IMDSv1.
- Tạo, gắn, mount, snapshot và resize EBS volume trên Linux thành thạo.
- Cấu hình thành công **Session Manager** truy cập EC2 không cần mở port SSH, đây là best practice mình muốn áp dụng xuyên suốt quá trình thực tập.
- Đọc và debug log hệ thống Linux thành thạo hơn.

### Khó khăn và bài học:

- Lúc đầu cấu hình Session Manager không kết nối được vì quên chưa gắn IAM role có policy `AmazonSSMManagedInstanceCore` cho EC2. Sau khi gắn role thì chạy ngon.
- Mount EBS volume lần đầu hay quên `mkfs.ext4` trước khi mount, dẫn đến lỗi `wrong fs type`.
- Phân biệt `systemctl` và `service` còn lúng túng, sau này mình chốt chỉ dùng `systemctl` cho các service hiện đại.