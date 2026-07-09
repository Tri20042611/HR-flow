---
title : "Lambda FileValidator & Extract"
date : 2024-01-01
weight : 4
chapter : false
---

### Lambda FileValidator

Lambda này được trigger bởi EventBridge mỗi khi có file mới upload vào S3 Quarantine.

**Luồng xử lý:**

```
Event S3 ObjectCreated
    ↓
_head_object → lấy metadata (job_id, channel)
    ↓
get_object → đọc file bytes
    ↓
_detect_mime_by_magic → kiểm tra magic bytes (chống đổi extension)
    ↓
Kiểm tra kích thước ≤ 5MB
    ↓
SHA-256 hash → query DynamoDB GSI sha256_hash-index
    ↓
PASS: copy sang Clean bucket + xóa Quarantine
FAIL: log + SNS alert HR (file giữ lại Quarantine 7 ngày)
```

**Code chính — kiểm tra MIME bằng magic bytes:**

```python
# src/file_validator/app.py

MAGIC_BYTES = {
    b'%PDF': 'application/pdf',
    b'PK\x03\x04': 'application/vnd.openxmlformats-officedocument.wordprocessingml.document',  # .docx
    b'\xd0\xcf\x11\xe0': 'application/msword',  # .doc (Compound Document)
}

def _detect_mime_by_magic(file_bytes: bytes) -> str | None:
    """Magic bytes không tin Content-Type header — chống đổi extension giả."""
    stripped = file_bytes.lstrip(b' \t\r\n\x00\x0b\x0c\xef\xbb\xbf')
    for magic, mime in MAGIC_BYTES.items():
        if stripped[:len(magic)] == magic:
            return mime
    return None
```

**Điểm quan trọng:** Dùng magic bytes thay vì `Content-Type` header vì header có thể bị spoof. Ví dụ: file `.exe` đổi thành `.pdf` vẫn có header `application/pdf` nhưng magic bytes sẽ cho thấy đây không phải PDF thật.

**Code — deduplication bằng SHA-256:**

```python
# src/file_validator/app.py

def _is_duplicate(sha256_hash: str) -> bool:
    """Query DynamoDB GSI trên sha256_hash để phát hiện file trùng lặp."""
    table = dynamodb.Table(CANDIDATES_TABLE)
    response = table.query(
        IndexName='sha256_hash-index',
        KeyConditionExpression='sha256_hash = :h',
        ExpressionAttributeValues={':h': sha256_hash},
        Limit=1
    )
    return len(response.get('Items', [])) > 0
```

**Validation fail → SNS alert HR:**

```python
def _handle_validation_fail(bucket_name, object_key, job_id, reason):
    """File không qua kiểm tra → alert HR, giữ lại Quarantine."""
    _send_hr_alert(
        subject='[HireFlow] File CV bị từ chối',
        message=f'Lý do: {reason}\nKey: {object_key}\nJob ID: {job_id}'
    )
    # File giữ trong Quarantine — lifecycle 7 ngày tự xóa
```

### Lambda Extract

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
