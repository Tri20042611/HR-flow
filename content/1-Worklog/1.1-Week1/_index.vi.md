---
title: "Worklog Tuần 1"
date: 2026-04-20
weight: 1
chapter: false
pre: " <b> 1.1. </b> "
---

### Mục tiêu tuần 1:

- Làm quen với các thành viên, mentor và nội quy của chương trình First Cloud AI Journey.
- Nắm tổng quan về AWS, các nhóm dịch vụ cốt lõi (Compute, Storage, Networking, Database, Security).
- Tạo tài khoản AWS Free Tier, làm quen với AWS Management Console và cài đặt AWS CLI.

> Tuần đầu tiên nên lịch làm việc không dày, mình chỉ có 3 buổi thực sự vào Thứ 2, Thứ 4, Thứ 6. Mấy ngày còn lại dành để đọc tài liệu ở nhà và xử lý việc riêng.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Buổi khởi động: <br> - Giới thiệu bản thân với các thành viên trong team, nhận mentor hỗ trợ <br> - Đọc & ký cam kết bảo mật thông tin (NDA), nắm các quy định khi thực tập tại AWS <br> - Được giới thiệu tổng quan lộ trình 12 tuần của chương trình | 20/04/2026 | 20/04/2026 | |
| 4 | - Làm quen AWS thực tế: <br> - Tạo tài khoản AWS Free Tier, bật MFA cho root <br> - Tạo IAM user riêng thay vì dùng root cho thao tác hằng ngày <br> - Cài AWS CLI v2, chạy `aws configure` với region `ap-southeast-1` <br> - Verify bằng `aws sts get-caller-identity` | 22/04/2026 | 22/04/2026 | <https://000001.awsstudygroup.com/> |
| 6 | - EC2 đầu tiên: <br> - Tìm hiểu EC2 cơ bản: instance type (`t2.micro`), AMI (Ubuntu 22.04), EBS mặc định 8GB <br> - Tạo key pair `.pem`, phân quyền 400 trên Linux <br> - Cấu hình Security Group chỉ mở port 22 từ IP cá nhân <br> - SSH thành công vào EC2, cài thử Nginx <br> - Đặt Budget alert ở mức 1 USD để chặn phát sinh ngoài ý muốn | 24/04/2026 | 24/04/2026 | <https://000004.awsstudygroup.com/> |

### Kết quả đạt được tuần 1:

- Hoàn tất onboarding chương trình, nắm được cơ chế làm việc 3 buổi/tuần với mentor và quy trình nộp worklog.
- Có cái nhìn bức tranh toàn cảnh về hệ sinh thái AWS, hiểu mỗi nhóm dịch vụ giải quyết bài toán gì (Compute chạy app, Storage lưu trữ, Networking kết nối, Database chứa dữ liệu có cấu trúc, Security kiểm soát truy cập).
- Tự tay đăng ký AWS Free Tier, bật MFA cho root account và tạo IAM user riêng — hiểu lý do phải tách root khỏi thao tác nghiệp vụ từ ngày đầu.
- Cài AWS CLI v2 trên máy cá nhân, chạy được lệnh `aws sts get-caller-identity` xác nhận credential hoạt động — đây là bước đệm để mọi thao tác sau này có thể tự động hoá bằng script.
- Khởi tạo EC2 đầu tiên thành công từ A-Z: chọn AMI, tạo key pair, cấu hình Security Group, SSH bằng key `.pem`, cài Nginx và truy cập được từ trình duyệt.
- Lần đầu đặt AWS Budget ở mức 1 USD và làm quen với Billing dashboard — hình thành thói quen kiểm tra chi phí sau mỗi buổi thực hành.

### Khó khăn và bài học:

- **Nhầm lẫn giữa Region và Availability Zone**: lúc đầu mình tưởng chọn `ap-southeast-1` là xong, sau đó tạo EC2 mới nhận ra phải chọn AZ cụ thể (`ap-southeast-1a`) khi launch instance. Hiểu hơn về kiến trúc Region → AZ → Subnet.
- **Quên chmod 400 cho key pair**: copy file `.pem` từ máy Windows sang Linux bị permission mặc định 644, SSH báo lỗi `UNPROTECTED PRIVATE KEY FILE`. Sau này luôn chạy `chmod 400 key.pem` ngay sau khi tải về.
- **Sốc với bảng giá AWS**: ban đầu nghĩ Free Tier là "free hoàn toàn", thực tế chỉ free trong giới hạn nhất định và một số dịch vụ (như Elastic IP khi không gắn vào instance chạy) vẫn tính phí theo giờ. Bài học: luôn terminate tài nguyên sau khi test xong, không để EC2 chạy qua đêm.
- **Chưa quen với thuật ngữ tiếng Anh**: những từ như `instance type`, `AMI`, `Security Group`, `EBS volume` đầu tuần khá lạ, sau vài lần thực hành thì nhớ và bắt đầu hình dung được từng khái niệm tương ứng với thành phần nào trên console.