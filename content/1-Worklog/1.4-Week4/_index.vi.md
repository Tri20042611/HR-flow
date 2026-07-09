---
title: "Worklog Tuần 4"
date: 2026-05-11
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

### Mục tiêu tuần 4:

- Hiểu sâu về **Amazon VPC**: CIDR, subnet (public/private), route table, Internet Gateway, NAT Gateway.
- Thực hành dựng VPC multi-tier (web tier public, app + db tier private) và đi gói tin từ ngoài vào trong.
- Tìm hiểu Security Group vs NACL, VPC Peering, Route 53, và các kết nối ra ngoài (VPC Endpoint, PrivateLink).

> Đây là tuần nặng nhất từ đầu chương trình với 5 buổi liên tiếp vì VPC là nền tảng của mọi thứ. Mình dành nhiều thời gian debug route table và security group.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Tổng quan VPC: CIDR block, VPC là gì trong AWS, khác gì với subnet <br> - Phân biệt public subnet và private subnet <br> - Khởi tạo VPC đầu tiên bằng wizard `VPC and more`, kiểm tra các resource được tạo tự động | 11/05/2026 | 11/05/2026 | <https://000003.awsstudygroup.com/> |
| 3 | - Tìm hiểu **Internet Gateway (IGW)** và **Route Table**: gói tin đi từ Internet vào public subnet đi qua IGW như thế nào <br> - Tạo 2 public subnet ở 2 AZ khác nhau <br> - Cấu hình route table cho public subnet trỏ `0.0.0.0/0` về IGW <br> - Launch EC2 ở public subnet, verify được truy cập được Internet qua `curl ifconfig.me` | 12/05/2026 | 12/05/2026 | <https://000003.awsstudygroup.com/> |
| 4 | - Tìm hiểu **NAT Gateway** và **NAT Instance**: tại sao private subnet cần NAT để ra Internet <br> - Tạo private subnet ở 2 AZ <br> - Triển khai NAT Gateway ở public subnet, route `0.0.0.0/0` từ private subnet về NAT <br> - Thực hành: EC2 ở private subnet dùng NAT để `apt update` nhưng không thể SSH từ ngoài vào | 13/05/2026 | 13/05/2026 | <https://000003.awsstudygroup.com/> |
| 5 | - Phân biệt **Security Group (stateful)** và **Network ACL (stateless)** <br> - Viết NACL custom: deny port 22 nhưng cho phép port 80, 443 <br> - Cài Apache HTTP Server + MariaDB trên 3 EC2 theo mô hình web - app - db <br> - Test: web tier truy cập được Internet, app tier chỉ nhận traffic từ web tier, db tier chỉ nhận traffic từ app tier | 14/05/2026 | 14/05/2026 | <https://000003.awsstudygroup.com/> |
| 6 | - Tìm hiểu **VPC Peering** và **VPC Endpoint** <br> - Tạo Interface VPC Endpoint cho S3, kiểm tra EC2 ở private subnet truy cập S3 mà không qua NAT Gateway <br> - Tìm hiểu DNS trong AWS: **Route 53** hosted zone, record type A, CNAME, alias <br> - Dọn dẹp toàn bộ VPC để tránh phát sinh phí | 15/05/2026 | 15/05/2026 | <https://000010.awsstudygroup.com/> |

### Kết quả đạt được tuần 4:

- Nắm vững mô hình VPC multi-tier và đi gói tin khi user truy cập từ ngoài vào web → app → db.
- Tự tay thiết kế được CIDR block cho VPC và subnet, biết chừa dải IP cho tương lai.
- Cấu hình thành công **Internet Gateway**, **NAT Gateway**, route table cho public/private subnet.
- Phân biệt rõ **Security Group (stateful)** và **Network ACL (stateless)**, biết khi nào nên dùng cái nào.
- Triển khai được **VPC Endpoint** (Interface endpoint cho S3) để giảm chi phí NAT và bảo mật traffic nội bộ.
- Nắm cơ bản **Route 53** hosted zone và các record type thường dùng (A, AAAA, CNAME, Alias).

### Khó khăn và bài học:

- Lúc đầu vẽ nhầm route table cho private subnet trỏ thẳng ra IGW thay vì NAT, khiến EC2 private bị lộ IP public. Có bài học rõ ràng về tầm quan trọng của **nguyên tắc tối thiểu (least privilege)** trong network.
- NAT Gateway tốn phí theo giờ, phải nhớ xóa sau khi thực hành để không phát sinh chi phí ngoài ý muốn.
- NACL stateless nên phải cho phép cả **inbound lẫn outbound** với ephemeral port, đây là điểm dễ nhầm so với Security Group.
