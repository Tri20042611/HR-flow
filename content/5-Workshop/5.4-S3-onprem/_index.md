---
title : "Lambda FileValidator & Extract"
date : 2024-01-01
weight : 4
chapter : false
---

### Lambda FileValidator

This Lambda is triggered by EventBridge each time a new file is uploaded to S3 Quarantine.

**Processing flow:**

```
S3 ObjectCreated Event
    ↓
_head_object → get metadata (job_id, channel)
    ↓
get_object → read file bytes
    ↓
_detect_mime_by_magic → check magic bytes (prevent extension spoofing)
    ↓
Check size ≤ 5MB
    ↓
SHA-256 hash → query DynamoDB GSI sha256_hash-index
    ↓
PASS: copy to Clean bucket + delete from Quarantine
FAIL: log + SNS alert HR (file kept in Quarantine 7 days)
```

**Key code — MIME checking via magic bytes:**

```python
# src/file_validator/app.py

MAGIC_BYTES = {
    b'%PDF': 'application/pdf',
    b'PK\x03\x04': 'application/vnd.openxmlformats-officedocument.wordprocessingml.document',  # .docx
    b'\xd0\xcf\x11\xe0': 'application/msword',  # .doc (Compound Document)
}

def _detect_mime_by_magic(file_bytes: bytes) -> str | None:
    """Magic bytes don't trust Content-Type header — prevents fake extension."""
    stripped = file_bytes.lstrip(b' \t\r\n\x00\x0b\x0c\xef\xbb\xbf')
    for magic, mime in MAGIC_BYTES.items():
        if stripped[:len(magic)] == magic:
            return mime
    return None
```

**Important:** Use magic bytes instead of `Content-Type` header because headers can be spoofed. Example: `.exe` file renamed to `.pdf` still has header `application/pdf` but magic bytes will reveal it's not a real PDF.

**Code — deduplication via SHA-256:**

```python
# src/file_validator/app.py

def _is_duplicate(sha256_hash: str) -> bool:
    """Query DynamoDB GSI on sha256_hash to detect duplicate files."""
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
    """File failed validation → alert HR, keep in Quarantine."""
    _send_hr_alert(
        subject='[HireFlow] CV file rejected',
        message=f'Reason: {reason}\nKey: {object_key}\nJob ID: {job_id}'
    )
    # File stays in Quarantine — lifecycle 7 days auto-delete
```

### Lambda Extract

Lambda is triggered by S3 notification each time a file lands in Clean bucket. Synchronous processing: start Textract job → poll until complete → save raw text → send SQS.

**Code — Textract async polling:**

```python
# src/extract/app.py

TEXTRACT_MAX_POLLS = 60       # 60 × 5s = 5 minutes timeout
TEXTRACT_POLL_INTERVAL = 5    # seconds

def _run_textract(bucket: str, key: str) -> str:
    # Step 1: Start async job
    response = textract.start_document_text_detection(
        DocumentLocation={'S3Object': {'Bucket': bucket, 'Name': key}}
    )
    job_id = response['JobId']

    # Step 2: Poll until SUCCEEDED or FAILED
    for attempt in range(TEXTRACT_MAX_POLLS):
        time.sleep(TEXTRACT_POLL_INTERVAL)
        result = textract.get_document_text_detection(JobId=job_id)
        status = result['JobStatus']

        if status == 'SUCCEEDED':
            return _extract_text_from_textract(job_id)
        if status == 'FAILED':
            raise RuntimeError(f'Textract FAILED: {result.get("StatusMessage")}')

    raise TimeoutError(f'Textract timeout after {TEXTRACT_MAX_POLLS * TEXTRACT_POLL_INTERVAL}s')
```

**Idempotency check — avoid duplicate processing:**

```python
# src/extract/app.py

def _process_file(clean_bucket: str, s3_key: str):
    # Check RIGHT AWAY — avoid duplicate when Lambda re-triggers
    existing = _check_already_processed(s3_key)
    if existing:
        print(f'[WARN] File already processed — candidate_id: {existing}')
        return

    # ... continue normal processing
```

**Text length validation — detect low-quality scanned PDF:**

```python
MIN_TEXT_LENGTH = 200  # characters

if len(raw_text.strip()) < MIN_TEXT_LENGTH:
    _update_candidate_status(candidate_id, 'extraction_failed')
    _send_hr_alert(
        subject='[HireFlow] CV content unreadable',
        message=f'Textract extracted < {MIN_TEXT_LENGTH} characters. '
                f'Possibly a low-quality scanned PDF.'
    )
    return
```
