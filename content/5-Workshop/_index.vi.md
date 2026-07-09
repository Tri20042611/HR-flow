---
title: "Workshop"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Deploy Backend - Pipeline xử lý CV tự động

![Workshop](/images/5-Workshop/workshop-header.png)

#### Tổng quan

Workshop này hướng dẫn xây dựng đường ống xử lý CV tự động hoàn toàn sử dụng các dịch vụ AWS Serverless. Từ lúc ứng viên upload CV lên S3, qua các bước xác thực, trích xuất văn bản bằng Amazon Textract, chấm điểm bằng LLM, cho đến khi kết quả được lưu vào DynamoDB và thông báo cho ứng viên.

**Kiến trúc:** Queue-based serverless pipeline, tách rời từng giai đoạn bằng SQS để đảm bảo scalability và fault-tolerance. Không có server persistent — toàn bộ Lambda execution, auto-scale theo số lượng CV.

#### Nội dung

1. [Tổng quan về workshop](5.1-Workshop-overview/)
2. [Prerequisites](5.2-Prerequiste/)
3. [Cấu trúc Project & Triển khai SAM](5.3-S3-vpc/)
4. [Lambda FileValidator & Extract](5.4-S3-onprem/)
5. [Lambda Score & Save&Notify](5.5-Policy/)
6. [Validation & Cleanup](5.6-Cleanup/)
7. [Challenges & Future](5.7-Challenges/)
8. [Thực nghiệm sản phẩm](5.8-ThucNghiepSanPham/)
