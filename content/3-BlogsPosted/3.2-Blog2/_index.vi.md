---
title: "Blog 2"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.2. </b> "
---

# SAM – Giúp mình deploy AWS mà không phải click chuột

Bữa trước mình có kể về Kiro rồi, cảm ơn mọi người đã quan tâm bài viết đó. Hôm nay kể tiếp về một thứ mà từ lúc dùng mình thấy đỡ cực hơn rất nhiều: SAM. Với mình thì đây đúng kiểu "bầu trời mới" của một thằng newbie.

## SAM là gì?

Nói đơn giản thì SAM (Serverless Application Model) là công cụ để viết Infrastructure as Code cho các dịch vụ serverless của AWS như Lambda, API Gateway, S3, DynamoDB, SQS, EventBridge... Có thể hình dung nó giống Terraform nhưng được AWS làm riêng cho hệ sinh thái của AWS.

Ai muốn đọc chi tiết thì xem tại đây:
https://docs.aws.amazon.com/serverless-application-model/

Hồi mới học AWS, chưa biết Kiro cũng chưa biết SAM, mọi thứ mình đều deploy bằng AWS Console.

- Click tạo Lambda.
- Click tạo S3.
- Click gắn trigger.
- Click sửa IAM.
- Click deploy.

Làm vài lần thì còn vui, chứ project có nhiều service là bắt đầu thấy "đấm vào thời gian". Có hôm chỉ sửa một Lambda mà ngồi click gần cả chục phút, lỡ quên một bước là lại làm lại từ đầu.

Lúc đó mình cũng đang làm một project nhỏ. Nhìn lại thì code còn lởm quá nên không dám nói tường tận cho mọi người, nhưng thời gian dành cho việc deploy còn nhiều hơn thời gian viết code.

Cho đến khi biết Kiro và SAM. giống hổ mọc thêm cánh.

## Điều mình thích nhất ở SAM là workflow

Workflow mình đang dùng:

1. Viết `template.yaml` để khai báo toàn bộ hạ tầng: Lambda, S3, DynamoDB, SQS, EventBridge...
2. `sam build` để SAM chuyển template thành CloudFormation.
3. `sam deploy` để deploy lên AWS, đồng thời tạo Change Set giúp review những gì sắp thay đổi trước khi apply.

![SAM Workflow](/images/3-BlogsPosted/3.2-Blog2/sam_workflow.png)

Cảm giác từ chỗ phải click từng service chuyển thành chỉ cần sửa file YAML rồi chạy vài câu lệnh.

## Điểm thứ hai mình rất thích là SAM Policies

Thay vì phải viết một đống IAM Policy JSON dài ngoằng như:

```json
{
  "Effect": "Allow",
  "Action": [
    "s3:PutObject",
    "s3:GetObject"
  ],
  "Resource": "arn:aws:s3:::my-bucket/*"
}
```

Thì boom:

```yaml
Policies:
  - S3CrudPolicy:
      BucketName: !Ref MyBucket
```

Nhìn phát là hiểu luôn, vừa ngắn vừa đỡ phải nhớ đủ loại ARN.

## Mấy cái "đau đầu" khi dùng SAM

### Circular Dependency

Đây chắc là cú tát đầu tiên mình gặp.

Mình mất nguyên một buổi chiều mới hiểu nó đang chửi cái gì (thật ra là AI giải thích cho mới ngộ ra 😅).

Case của mình là:

- S3 Bucket cần trigger Lambda khi có file upload.
- Nhưng Lambda lại reference chính Bucket đó để đọc file.
- CloudFormation nhìn vào thì không biết phải tạo Bucket trước hay Lambda trước.

Thế là báo:

> "Circular dependency between resources."

Lúc mới nhìn lỗi thì kiểu: "Ủa liên quan gì nhau?"

Hiểu được nguyên nhân rồi thì mới thấy CloudFormation cũng có lý.

### Vibe Coding làm mình... mất não

Có SAM + AI Agent thì đúng là sướng thật.

Mình chỉ cần mô tả yêu cầu, AI viết gần hết.

Nhưng cũng vì thế mà nhiều lúc nhìn lại project do chính mình làm mà phải ngồi đọc từ đầu mới hiểu nó đang chạy kiểu gì.

Tiết kiệm thời gian thì có.

Nhưng nếu lạm dụng quá thì cũng hơi đáng quan ngại thật.

## Theo mình thì ai nên dùng SAM?

- Làm backend trên AWS.
- Dùng nhiều Lambda hoặc nhiều service serverless.
- Đang dùng Kiro hoặc AI Agent để code thì SAM gần như sinh ra để AI đọc và chỉnh sửa.

Mình vẫn còn là newbie với AWS nên chắc chắn còn rất nhiều thứ chưa biết. Nếu anh em có kinh nghiệm với SAM, CloudFormation hay Serverless thì rất mong được góp ý để mình học thêm.
