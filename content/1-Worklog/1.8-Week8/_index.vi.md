---
title: "Worklog Tuần 8"
date: 2026-06-08
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---

### Mục tiêu tuần 8:

- Triển khai ứng dụng **serverless** với Lambda + API Gateway + DynamoDB, hiểu cơ chế event-driven.
- Làm quen **AWS CloudFormation**: viết template, deploy, update và rollback stack.
- Tổng kết toàn bộ 8 tuần thực tập, xác định hướng cho đồ án cuối khóa.

> Tuần cuối của giai đoạn nền tảng, mình chỉ có 3 buổi làm việc vì dành thời gian chuẩn bị đề xuất đồ án và tham gia buổi chia sẻ cuối khóa với mentor.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Tổng quan **AWS Lambda**: function, trigger, execution role, runtime <br> - Viết function Lambda bằng Python 3.12 trả về JSON đơn giản <br> - Tạo **API Gateway REST API** trỏ vào Lambda, test bằng `curl` <br> - Kết nối Lambda với **DynamoDB Streams** đã setup từ tuần 5, ghi mỗi thay đổi vào S3 | 08/06/2026 | 08/06/2026 | <https://000078.awsstudygroup.com/vi/2-resize-image-function/> |
| 4 | - Tìm hiểu **AWS CloudFormation**: template, stack, resource, parameter, output <br> - Viết template triển khai VPC 2-tier (public + private subnet, IGW, route table) <br> - Deploy stack, kiểm tra resource được tạo đúng <br> - Update stack (đổi CIDR), quan sát CloudFormation sửa theo changeset | 10/06/2026 | 10/06/2026 | <https://docs.aws.amazon.com/cloudformation/> |
| 6 | - Viết template CloudFormation triển khai Lambda + API Gateway + DynamoDB theo kiến trúc serverless tuần này <br> - Xóa resource tạo thủ công, deploy lại bằng stack để chứng minh tính tái sử dụng <br> - Thử cố tình tạo lỗi (conflict tên S3 bucket) để quan sát CloudFormation **rollback** <br> - Buổi tổng kết với mentor, ghi nhận feedback cho đồ án cuối khóa | 12/06/2026 | 12/06/2026 | <https://docs.aws.amazon.com/cloudformation/> |

### Kết quả đạt được tuần 8:

- Tạo và deploy được Lambda function, biết cách gắn **execution role** đúng nguyên tắc least privilege.
- Kết nối API Gateway + Lambda + DynamoDB thành một **REST API serverless** hoàn chỉnh, test thành công qua `curl`.
- Tái sử dụng **DynamoDB Streams** + Lambda + S3 từ tuần 5 để xây dựng luồng event-driven.
- Viết được template **CloudFormation** triển khai VPC 2-tier, nắm được cơ chế stack, parameter, output.
- Quan sát được CloudFormation **rollback** khi có lỗi, hiểu được lý do nên dùng IaC thay vì click tay trong console.
- Hoàn thành **vòng nền tảng 8 tuần**, sẵn sàng cho 4 tuần tiếp theo tập trung vào đồ án cuối khóa.

### Khó khăn và bài học:

- Lambda execution role mặc định không có quyền ghi vào CloudWatch Logs ở account mới, mình phải tự gắn policy `AWSLambdaBasicExecutionRole`.
- API Gateway cần **deploy stage** mới có URL gọi được, đây là bước dễ quên vì console không cảnh báo.
- CloudFormation rollback chỉ áp dụng khi bật rollback trong stack option; nếu không, resource đã tạo trước đó sẽ giữ lại gây trạng thái "nửa vời" - mình đã rơi vào tình huống này và mất 30 phút dọn dẹp tay.