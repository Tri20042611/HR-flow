---
title: "Blog 1"
date: 2026-07-06
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# Mình vừa nghịch Kiro được 2 tuần – kể bạn nghe cho vui

Bạn nào đang dùng Cursor hay Copilot rồi thì ngồi xuống đây, có 1 ông mới mà mình thấy khá hay ho, tên là Kiro – do AWS làm ra. Mình không PR gì cho ai nha, mình chỉ kể đại thôi vì thấy nó "lạ" so với mấy ông AI code kia.

## Ông Kiro này là gì vậy?

Nói nôm na: Cursor hay Copilot là "chat với AI rồi nó ra code". Còn Kiro thì nó bắt bạn ngồi viết kịch bản trước đã – kiểu như lên kế hoạch cưới vợ vậy, phải có spec mới cho làm.

Thật ra lúc đầu mình cũng chửi thầm: "làm cái demo nhỏ mà bắt viết giấy tờ chi cho mệt?". Nhưng xài được 1 tuần mình mới hiểu – đúng là viết trước rồi mới làm nó đỡ đập phá hơn thật. Hồi xài Cursor mình code được nửa ngày rồi nhận ra "à, chọn sai database mẹ rồi" – quay đầu đập hết. Kiro thì nó đẩy cái đau đó lên sớm, trong lúc viết spec, sửa rẻ hơn nhiều.

## Workflow nó bắt mình làm thế này

**Bước 1:** Mình gõ "Làm cho tao cái chức năng đăng nhập bằng Google" bằng tiếng Việt luôn cũng được.

**Bước 2:** Nó sinh ra 1 file `requirements.md` – kiểu liệt kê user phải làm gì, điều gì thì pass. Nhìn xong tự nhiên thấy mình mơ hồ chỗ nào luôn á.

**Bước 3:** Nó sinh tiếp `design.md` – kiểu sơ đồ kiến trúc, database gì, API gì, lỗi thì xử lý sao. Mình đọc, sửa, duyệt.

**Bước 4:** Nó chia nhỏ ra thành `tasks.md` – mỗi task 1 việc, có đánh dấu cái nào bắt buộc cái nào optional cho mình chọn.

**Bước 5:** Nó code từng task một, tự tick xong cái nào xong cái đó. Xong xuôi thì commit luôn.

Điểm mình thích là: sửa spec là nó tự update code theo. Hôm trước mình thêm 2 dòng vào `requirements.md` kiểu "à quên, phải có tính năng tag", 5 phút sau nó đã update cả design + tasks + code cho mình. Ngồi đó mình còn không kịp uống ngụm nước.

## Mấy thứ hay ho mà mình đang xài

**Steering files** – đây là thứ mình ước có hồi xài Cursor. Bạn viết 1 file markdown, set luật chơi cho AI, nó tuân theo mọi session. Kiểu mình viết "Lambda phải có timeout dưới 30s, S3 phải bật versioning, UI tiếng Việt nhưng code identifier tiếng Anh" – nó nhớ hết, không phải nhắc lại. File nào dùng luật nào thì gắn thêm cái điều kiện vào, mấy file khác không liên quan thì nó không load, đỡ ngốn.

**Hooks** – bạn cài trigger kiểu "save file TypeScript thì tự chạy eslint", "save template.yaml thì tự sam validate". Nó như có 1 ông senior ngồi review code mình ngay lúc save luôn. Đặc biệt cái này không tính quota – mấy cái check lặt vặt chạy thoải mái.

**MCP** – cắm mấy cái server vào thì Kiro gọi được AWS thật. Mình hay hỏi "Lambda này đang lỗi gì" – nó tự mò vô CloudWatch logs kéo log ra cho mình. Hay "S3 nào đang public" – nó gọi `s3:GetBucketPublicAccessBlock` trả kết quả luôn. Không phải nhảy qua console, đỡ mất não.

## Mấy cái dở mình gặp

Free tier 50 lần chat mỗi tháng – xài cho vui thôi chứ làm dự án thật không đủ. Mình đang trial bản Pro $20.

Nó chậm hơn Cursor cho mấy cái nhỏ nhỏ. Kiểu muốn "demo 30 phút cho sếp xem" thì chat mode Cursor vẫn nhanh hơn.

Quen rồi sẽ thấy dễ chịu nhưng nó vẫn trẻ, có lúc nó vẫn lú. Mình sửa code tay xong quên bảo nó update spec thì 2 hôm sau nó code chồng lên của mình luôn – phải nhớ refresh.

## Ai thích hợp dùng cái này?

Theo mình thì:

- **Bạn nào làm backend AWS, IaC, mấy dự án nhiều service** – Kiro sinh ra cho bạn đó. Spec + steering + hooks là combo rất ngon (ý kiến riêng thôi vì mình cũng newbie mảng này).
- **Bạn nào chỉ muốn demo nhanh cho bạn bè xem, làm MVP 1 buổi** – cứ ở lại với Cursor đi, đừng ép Kiro.
- **Team mà muốn ép mọi người cùng 1 coding style, cùng 1 chuẩn bảo mật** – steering files của Kiro là giải pháp luôn, không phải dán rule vô README rồi ai cũng quên.

## Setup thì sao?

Tải về từ kiro.dev, tạo 1 folder `.kiro/steering/` bỏ vài file markdown luật chơi vô, tạo folder `.kiro/hooks/` bỏ vài file JSON trigger vô, xong. 10 phút là chạy được.

Còn đang phân vân thì cứ free tier mà nghịch. 50 lần đủ để bạn tự quyết có nên nâng cấp hay không.

## Kết

Mình không nói Kiro giết chết Cursor hay gì đó vì hai ông khác nhau. Mình chỉ nói: nếu bạn đang làm AWS nghiêm túc mà vẫn dùng AI như "chat rồi paste code" thì thiệt hơi phí. Kiro nó thay đổi cách bạn làm việc với AI, từ "hỏi rồi nhận" thành "cùng lên kế hoạch rồi mới làm".

## Chuyện ngoài lề: share ae duyên của tôi như thế nào với Kiro

Thật ra mình bén duyên với Kiro nhờ 1 anh Nghị vào buổi tối T7 đó là lần đầu mình biết tới nó và cách dùng. Về sau mình có xem các video. Vì lúc đó mình newbie chỉ nghĩ AWS thao tác chủ yếu trên web đó thôi, về sau mình cũng từ Kiro chỉ cho mình vùng trời mới về "SAM". Hẹn lần sau kể ae.

![Dashboard Kiro với 3 chế độ Spec, Vibe, Agent](/images/kiro-dashboard.png)
