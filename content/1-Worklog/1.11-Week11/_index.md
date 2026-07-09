---
title: "Week 11 Worklog"
date: 2026-06-29
weight: 11
chapter: false
pre: " <b> 1.11. </b> "
---

### Week 11 Objectives:

- Complete the **OCR → score → save** pipeline for HireFlow, going from raw CV PDF to a candidate row with a score in DynamoDB.
- Integrate **LLM scoring** (OpenAI-compatible API) to rate CVs against a Job Description.
- Send **confirmation email** to candidates via SES when their CV scores high enough and is saved successfully.

> This week has 4 sessions (skipping Thursday). Each session corresponds to one layer in the async pipeline, from OCR callback all the way to the final email.

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| Mon | - Write Lambda **extract-complete** to receive SNS callback from Textract (`JobStatus=SUCCEEDED`) <br> - Call `GetDocumentTextDetection` to retrieve full text from the Textract job <br> - Parse text → save to S3 `OCR Results` and push message `{candidate_id, job_id, cv_text}` into **SQS `score-queue`** <br> - Update DynamoDB row: status `extracting → extracted` | 29/06/2026 | 29/06/2026 | <https://docs.aws.amazon.com/textract/> |
| Tue | - Write Lambda **score** consumer from `score-queue`, using **Amazon Bedrock (Claude)** or OpenAI-compatible endpoint (`9router.xjanua.me/v1`, model `Tri_Bui`) <br> - Prompt with `JD + cv_text` → LLM returns JSON `{score, reasoning, strengths, gaps}` <br> - Validate JSON output (prevent LLM hallucination), push result to **SQS `save-queue`** if `score > threshold` (default 60) <br> - DDB status: `extracted → scoring → scored` (or `low_score` if below threshold) | 30/06/2026 | 30/06/2026 | <https://docs.aws.amazon.com/bedrock/> |
| Wed | - Write Lambda **save-notify** consumer from `save-queue`, update final DDB row with `score, reasoning, strengths, gaps`, status `scored → saved` <br> - Configure **Amazon SES**: verify domain/sender, request production access (sandbox first) <br> - Send email template to candidate: "Thank you for applying, your CV has been AI-screened..." <br> - Test with 1 real CV, check inbox (and spam folder) | 01/07/2026 | 01/07/2026 | <https://docs.aws.amazon.com/ses/> |
| Fri | - Configure **DLQ** for both `score-queue` and `save-queue`, set `maxReceiveCount = 3` <br> - Intentionally inject erroneous messages (missing `cv_text` field, or LLM returns unparseable JSON) <br> - Observe messages being retried 3 times then moved to DLQ <br> - Write Lambda **dlq-handler** (optional) to log detailed errors, alert HR via SNS when pipeline is failing <br> - Update `README.md` with ASCII architecture diagram and troubleshooting guide | 03/07/2026 | 03/07/2026 | <https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/> |

### Week 11 Achievements:

- Completed **async OCR → score → save pipeline**: CV PDF flows from S3 Quarantine to DynamoDB row with full `score, reasoning, strengths, gaps`.
- Successfully integrated **LLM scoring** via OpenAI-compatible endpoint, with output validation to prevent parse errors.
- **Amazon SES** sends confirmation email to candidates successfully right after their CV is scored and saved.
- Configured **DLQ** for both SQS queues, tested retry → DLQ flow for erroneous messages.
- Each candidate has a **UUID `candidate_id`** with a complete status flow: `uploaded → extracting → extracted → scoring → scored → saved` (or `low_score`, `extraction_failed`).

### Challenges and Lessons Learned:

- Textract SNS callback cannot be auto-bound to Lambda in the SAM template; had to create SNS topic + subscription manually using `aws sns subscribe` after deployment — documented in `scripts/setup_sns.sh` for repeatable setup.
- LLM sometimes wraps the output in ```json``` markdown fences causing parse failure; had to add regex stripping before `json.loads`. There were also cases where the model omitted the `gaps` field — lesson: always provide a default fallback for optional fields.
- SES in sandbox mode can only send to verified email addresses; testing with a personal Gmail account failed because the recipient was not verified — had to verify each recipient or request production access.
- LLM is a third-party endpoint (`9router.xjanua.me`) with a downtime risk — documented under "Known limitations" in README; production should self-host or use Bedrock instead.
