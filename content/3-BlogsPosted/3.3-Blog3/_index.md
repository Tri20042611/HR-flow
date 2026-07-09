---
title: "Blog 3"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.3. </b> "
---

# Automating PostgreSQL Audit Log Extraction and Analysis with Amazon S3

While exploring AWS, I came across an article about automating the audit log processing pipeline for Amazon RDS PostgreSQL and Aurora PostgreSQL.

## Background

When pgAudit is enabled on PostgreSQL, all SQL operations such as SELECT, INSERT, UPDATE, DELETE, and DDL are recorded. However, these logs sit in CloudWatch Logs as raw, unstructured text. Manual review or analysis is very time-consuming, especially for teams that need to meet regular security and auditing requirements.

## Solution

A fully automated pipeline based on an event-driven architecture with the following components:

1. PostgreSQL generates audit logs via pgAudit
2. CloudWatch Logs receives logs and uses a subscription filter to detect lines containing the "AUDIT" keyword
3. Triggers AWS Lambda for processing
4. Lambda parses raw text into structured CSV with all fields: timestamp, user, database, action, object_type, object_name, query
5. Saves to Amazon S3 with year/month/day partitioning
6. Optionally uses Amazon Athena to query directly with SQL

![pgAudit Pipeline Architecture](/images/3-BlogsPosted/3.3-Blog3/pgaudit_pipeline.png)

The entire infrastructure is packaged in a single CloudFormation template, creating an S3 bucket with encryption and versioning, least-privilege IAM role, Python 3.12 Lambda function, and subscription filter. Deploy time is approximately 2–3 minutes.

## Cost considerations

The most notable cost is **CloudWatch Logs ingestion at $0.50/GB**, as it scales directly with the number of SQL statements being audited. Lambda and S3 are usually not a concern because CloudWatch subscription filters batch multiple log events into a single invoke, making the cost for these two services very low.

S3 lifecycle policy automatically transitions old logs to Standard-IA after 90 days and Glacier after 1 year to optimize long-term storage costs.

If auditing every statement is not needed, you can change the config to `pgaudit.log = 'ddl, role'` instead of `all` to significantly reduce log volume.

## Security considerations

Since the pipeline stores the full SQL statement content including parameters, it may contain sensitive data such as personally identifiable information (PII). Recommendations:

- Restrict S3 bucket access
- Consider using Amazon Macie to scan for sensitive data
- Lambda can be extended to mask or redact parameters before writing to S3

## Conclusion

This is a practical solution that addresses a real problem many teams face: logs exist but are not effectively usable. This pipeline transforms audit logs from unreadable text into structured, queryable CSV via Athena, with low cost and no complex infrastructure investment.

**Reference:** https://aws.amazon.com/blogs/database/automate-postgresql-audit-log-extraction-and-analysis-with-amazon-s3/
