---
title: "Week 9 Worklog"
date: 2026-06-15
weight: 9
chapter: false
pre: " <b> 1.9. </b> "
---

### Week 9 Objectives:

- Get acquainted with **Kiro** (AI IDE supporting code generation, refactoring, and test writing) to speed up personal workflow.
- Explore **Amazon SQS**: standard queue, visibility timeout, long polling, Dead-Letter Queue.
- Brainstorm architecture for the **final project**: HireFlow AI — a serverless recruitment system that helps HR screen CVs using AI.

> This week has 4 sessions: 1 tool familiarization (Kiro), 1 SQS session, 1 architecture brainstorming for HireFlow, and the final session to lock down the approach and assign tasks for the next 3 weeks.

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| Mon | - Install and configure **Kiro IDE** on local machine <br> - Explore modes: chat, agent, inline edit, spec-driven <br> - Write a simple Lambda function via Kiro prompt, compare output with hand-written code <br> - Use Kiro to refactor Python code from Week 8 (Lambda + DynamoDB) for cleaner structure | 15/06/2026 | 15/06/2026 | <https://kiro.dev/> |
| Tue | - Propose 2-3 final project ideas: <br>&emsp; + **HireFlow AI** — serverless recruitment platform, HR reviews CVs with AI (Textract OCR + LLM scoring + SES email) <br>&emsp; + Async order processing system with SQS + Lambda + DynamoDB <br>&emsp; + Log ingestion pipeline from CloudWatch → SQS → Lambda → S3/Redshift <br> - Analyze pros/cons of each option (cost, complexity, learning value) <br> - Lock in **HireFlow AI**, draw high-level architecture diagram (upload CV → validate → OCR → score → email) on Excalidraw <br> - Assign tasks for the remaining 3 weeks of the project | 16/06/2026 | 16/06/2026 | <https://cloudjourney.awsstudygroup.com/> |
| Thu | - **Amazon SQS** overview: standard queue vs FIFO queue, message size, retention <br> - Create SQS standard queue `orders-queue` via console (for learning; will rename to CV pipeline queue later) <br> - Write Lambda **producer** to push JSON messages into queue, Lambda **consumer** to read and write to DynamoDB <br> - Configure **visibility timeout**, **message retention**, **long polling (20s)** <br> - Test with a large batch (~500 messages) to observe consumer behavior | 18/06/2026 | 18/06/2026 | <https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/> |
| Fri | - Create **Dead-Letter Queue** `orders-dlq` for `orders-queue`, configure `maxReceiveCount = 3` <br> - Intentionally push erroneous messages to observe the flow: main queue → DLQ → Lambda handling separately <br> - Map SQS pattern into HireFlow architecture: 2 queues `score-queue` + `save-queue` sitting between `extract → score → save-notify` <br> - Complete project `README.md` for HireFlow: goals, architecture, technologies (8 Lambda, 6 S3, 2 DDB, SQS, Textract, SES), 3-week plan <br> - Wrap up Week 9, prepare for Week 10 (switch to using SAM) | 19/06/2026 | 19/06/2026 | <https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/> |

### Week 9 Achievements:

- Installed and become proficient with **Kiro IDE** in chat, agent, and inline edit modes; saved approximately 30% of coding time compared to hand-writing.
- Used Kiro to refactor Week 8 Lambda code; output is cleaner with full docstrings, verified with manual tests.
- Locked in **final project idea: HireFlow AI — Serverless Recruitment Platform** with high-level architecture diagram and task assignments for the next 3 weeks.
- Created and operated **SQS standard queue** with producer/consumer Lambda; understood visibility timeout and long polling — ready to apply to the extract→score→save pipeline in HireFlow.
- Configured **Dead-Letter Queue** and observed erroneous messages being routed to DLQ correctly after 3 retries.

### Challenges and Lessons Learned:

- Kiro generates code quickly but sometimes "hallucinates" — calling non-existent APIs or missing imports. Lesson: always review AI-generated code before running it for real.
- SQS consumer Lambda defaults to batch size = 10; initially forgot to tune it so throughput was low; increased batch size to 10 with concurrency 5 to handle 500 messages in time.
- Initially created the DLQ but forgot to set the `redrive policy` on the main queue, so failed messages kept sitting in the main queue instead of moving to DLQ — had to update the policy to fix it.
