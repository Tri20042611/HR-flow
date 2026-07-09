---
title: "Workshop"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Deploy Backend - Automated CV Processing Pipeline

![Workshop](/images/5-Workshop/workshop-header.png)

#### Overview

This workshop guides you through building a fully automated CV processing pipeline using AWS Serverless services. From candidate uploading CV to S3, through validation, text extraction via Amazon Textract, LLM-based scoring, to storing results in DynamoDB and notifying the candidate.

**Architecture:** Queue-based serverless pipeline, separating each stage with SQS for scalability and fault-tolerance. No persistent servers — all Lambda execution, auto-scaling based on CV volume.

#### Content

1. [Workshop overview](5.1-Workshop-overview/)
2. [Prerequisites](5.2-Prerequiste/)
3. [Project Structure & SAM Deploy](5.3-S3-vpc/)
4. [Lambda FileValidator & Extract](5.4-S3-onprem/)
5. [Lambda Score & Save&Notify](5.5-Policy/)
6. [Validation & Cleanup](5.6-Cleanup/)
7. [Challenges & Future](5.7-Challenges/)
8. [Product Demonstration](5.8-ThucNghiepSanPham/)
