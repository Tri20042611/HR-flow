---
title: "Worklog Week 5"
date: 2026-05-18
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

### Week 5 Goals:

- Deploy **Amazon RDS** (Multi-AZ, Read Replica, backup, parameter group) for a typical OLTP use case.
- Get hands-on with **Amazon DynamoDB**: partition key, sort key, GSI, throughput and auto scaling.
- Place databases in private subnets and secure them with Security Groups plus at-rest & in-transit encryption.

> A lighter week with 3 sessions: 2 for RDS, 1 for DynamoDB. Mid-week I also read the Aurora whitepaper to prepare for the final project.

### Tasks for this week:

| Day | Task | Start date | End date | Reference |
| --- | --- | --- | --- | --- |
| 2 | - Overview of managed databases on AWS: RDS, Aurora, DynamoDB, ElastiCache, Neptune, Redshift <br> - Created an RDS MySQL db.t3.micro in the private subnets of the VPC built in week 4 <br> - Configured the Security Group to allow only the app-tier EC2 to reach port 3306 <br> - Connected from the app-tier EC2 using the `mysql` client, created a schema and inserted sample data | 18/05/2026 | 18/05/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 4 | - Enabled **Multi-AZ Deployment** for automatic failover <br> - Enabled **automated backup** with 7-day retention, verified in AWS Backup <br> - Created a **cross-region Read Replica** for reporting <br> - Restored a **DB snapshot** into a new DB instance to verify recoverability <br> - Created a custom **DB Parameter Group**, changed `max_connections` and applied it | 20/05/2026 | 20/05/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 6 | - DynamoDB overview: NoSQL key-value, partition key, sort key, GSI/LSI <br> - Created a `Users` table with partition key `userId` and sort key `createdAt` <br> - Practiced CRUD via `aws dynamodb` CLI: `put-item`, `get-item`, `update-item`, `query`, `scan` <br> - Enabled **DynamoDB Streams** + a Lambda trigger to write change logs into S3 <br> - Configured **auto scaling** on read/write capacity and simulated traffic to watch capacity scale up | 22/05/2026 | 22/05/2026 | <https://cloudjourney.awsstudygroup.com/> |

### Week 5 Outcomes:

- Differentiated managed databases on AWS and chose the right one for each use case (RDS for relational OLTP, DynamoDB for large-scale NoSQL, ElastiCache for caching...).
- Deployed RDS MySQL in private subnets, connected successfully from the app-tier EC2.
- Enabled **Multi-AZ**, **cross-region Read Replica** and **automated backup**; able to restore from snapshot.
- Created and managed a **DynamoDB table**, used the CLI for CRUD, understood how partition/sort key shape query patterns.
- Wired **DynamoDB Streams** to Lambda + S3 for a durable event store.

### Difficulties and lessons learned:

- I first placed the RDS in a public subnet; after my mentor flagged it I moved it to the private subnet and restricted the SG to the app tier only. Best practice I will keep applying.
- DynamoDB `Scan` is expensive and resource-heavy — design partition keys so most queries use `Query` instead.
- Multi-AZ requires a subnet group spanning at least 2 AZs. I forgot to create the subnet group first, so Multi-AZ could not be enabled on the first attempt.