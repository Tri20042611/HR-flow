---
title: "Worklog Tuần 5"
date: 2026-05-18
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

### Mục tiêu tuần 5:

- Triển khai **Amazon RDS** (Multi-AZ, Read Replica, backup, parameter group) cho bài toán OLTP quen thuộc.
- Làm quen **Amazon DynamoDB**: partition key, sort key, GSI, throughput và auto scaling.
- Đặt DB trong private subnet, bảo mật bằng Security Group và mã hóa at-rest & in-transit.

> Tuần này lịch khá gọn với 3 buổi, mình dành 2 buổi cho RDS và 1 buổi cho DynamoDB. Giữa tuần có 1 buổi đọc whitepaper về Aurora để chuẩn bị cho đồ án cuối khóa.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Tổng quan managed database trên AWS: RDS, Aurora, DynamoDB, ElastiCache, Neptune, Redshift <br> - Tạo RDS MySQL db.t3.micro đầu tiên, đặt trong private subnet của VPC đã dựng tuần 4 <br> - Cấu hình Security Group chỉ cho phép EC2 app tier kết nối vào port 3306 <br> - Từ EC2 app tier dùng `mysql client` kết nối thành công, tạo schema và insert dữ liệu mẫu | 18/05/2026 | 18/05/2026 | <https://000005.awsstudygroup.com/> |
| 4 | - Bật **Multi-AZ Deployment** để tự động failover <br> - Bật **automated backup** giữa 7 ngày, kiểm tra trong AWS Backup <br> - Tạo **Read Replica** ở region khác (cross-region) phục vụ báo cáo <br> - Restore một **DB snapshot** ra một DB instance mới để kiểm tra khả năng khôi phục <br> - Tạo custom **DB Parameter Group**, đổi `max_connections` và áp dụng | 20/05/2026 | 20/05/2026 | <https://000005.awsstudygroup.com/> |
| 6 | - Tổng quan DynamoDB: NoSQL key-value, partition key, sort key, GSI/LSI <br> - Tạo bảng `Users` với partition key `userId` và sort key `createdAt` <br> - Thực hành CRUD với `aws dynamodb` CLI: `put-item`, `get-item`, `update-item`, `query`, `scan` <br> - Bật **DynamoDB Streams** + Lambda trigger để ghi log mỗi lần có thay đổi vào S3 <br> - Cấu hình **auto scaling** cho read/write capacity, mô phỏng tăng tải để xem capacity scale lên | 22/05/2026 | 22/05/2026 | <https://000078.awsstudygroup.com/> |

### Kết quả đạt được tuần 5:

- Phân biệt được các loại managed database trên AWS và chọn đúng cho từng bài toán (RDS cho OLTP quan hệ, DynamoDB cho NoSQL scale lớn, ElastiCache cho cache...).
- Triển khai RDS MySQL ở private subnet, kết nối qua EC2 app tier thành công.
- Bật được **Multi-AZ**, **Read Replica cross-region**, **automated backup** và biết restore từ snapshot.
- Tạo và quản lý **DynamoDB table**, dùng CLI thao tác CRUD, hiểu partition key và sort key ảnh hưởng đến query pattern.
- Kết nối **DynamoDB Streams** với Lambda và S3 để làm kho lưu trữ sự kiện lâu dài.

### Khó khăn và bài học:

- Lúc đầu đặt RDS ở public subnet, sau khi mentor review nhắc mình mới chuyển về private subnet và chỉ mở SG cho app tier. Đây là best practice mình sẽ áp dụng xuyên suốt.
- DynamoDB `Scan` rất tốn chi phí và tài nguyên, phải thiết kế partition key sao cho phần lớn query dùng `Query` thay vì `Scan`.
- Multi-AZ cần subnet group trải rộng ít nhất 2 AZ, mình quên tạo subnet group trước nên RDS không cho bật Multi-AZ ngay lần đầu.