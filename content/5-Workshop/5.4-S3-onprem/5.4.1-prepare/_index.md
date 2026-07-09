---
title : "Lambda Extract"
date : 2024-01-01
weight : 4
chapter : false
---

## 4. Lambda Extract — OCR with Amazon Textract

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
