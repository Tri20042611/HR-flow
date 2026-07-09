---
title : "Lambda Extract"
date : 2024-01-01
weight : 4
chapter : false
---

## 4. Lambda Extract — OCR với Amazon Textract

Lambda được trigger bởi S3 notification mỗi khi file mới vào Clean bucket. Xử lý đồng bộ (synchronous): start Textract job → poll đến khi hoàn thành → lưu raw text → gửi SQS.

**Code — Textract async polling:**

```python
# src/extract/app.py

TEXTRACT_MAX_POLLS = 60       # 60 × 5s = 5 phút timeout
TEXTRACT_POLL_INTERVAL = 5    # giây

def _run_textract(bucket: str, key: str) -> str:
    # Bước 1: Start async job
    response = textract.start_document_text_detection(
        DocumentLocation={'S3Object': {'Bucket': bucket, 'Name': key}}
    )
    job_id = response['JobId']

    # Bước 2: Poll đến khi SUCCEEDED hoặc FAILED
    for attempt in range(TEXTRACT_MAX_POLLS):
        time.sleep(TEXTRACT_POLL_INTERVAL)
        result = textract.get_document_text_detection(JobId=job_id)
        status = result['JobStatus']

        if status == 'SUCCEEDED':
            return _extract_text_from_textract(job_id)
        if status == 'FAILED':
            raise RuntimeError(f'Textract FAILED: {result.get("StatusMessage")}')

    raise TimeoutError(f'Textract timeout sau {TEXTRACT_MAX_POLLS * TEXTRACT_POLL_INTERVAL}s')
```

**Idempotency check — tránh xử lý trùng:**

```python
# src/extract/app.py

def _process_file(clean_bucket: str, s3_key: str):
    # Check NGAY LÚC ĐẦU — tránh duplicate khi Lambda tự trigger lại
    existing = _check_already_processed(s3_key)
    if existing:
        print(f'[WARN] File đã xử lý — candidate_id: {existing}')
        return

    # ... tiếp tục xử lý bình thường
```

**Text length validation — phát hiện PDF scan chất lượng thấp:**

```python
MIN_TEXT_LENGTH = 200  # ký tự

if len(raw_text.strip()) < MIN_TEXT_LENGTH:
    _update_candidate_status(candidate_id, 'extraction_failed')
    _send_hr_alert(
        subject='[HireFlow] CV không đọc được nội dung',
        message=f'Textract trích xuất < {MIN_TEXT_LENGTH} ký tự. '
                f'Có thể là PDF scan chất lượng thấp.'
    )
    return
```
