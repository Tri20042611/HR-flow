---
title: "Worklog Week 8"
date: 2026-06-08
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---

### Week 8 Goals:

- Deploy a **serverless** application with Lambda + API Gateway + DynamoDB and understand event-driven design.
- Get familiar with **AWS CloudFormation**: write templates, deploy, update and roll back stacks.
- Wrap up the entire 8-week foundation and outline the direction for the final project.

> Last week of the foundation phase. Only 3 working sessions because I spent the rest of the time preparing the project proposal and joining the wrap-up session with my mentor.

### Tasks for this week:

| Day | Task | Start date | End date | Reference |
| --- | --- | --- | --- | --- |
| 2 | - **AWS Lambda** overview: function, trigger, execution role, runtime <br> - Wrote a Python 3.12 Lambda function returning simple JSON <br> - Created a **REST API Gateway** pointing at the Lambda and tested with `curl` <br> - Connected Lambda to the **DynamoDB Streams** set up in week 5 to write every change into S3 | 08/06/2026 | 08/06/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 4 | - **AWS CloudFormation** overview: template, stack, resource, parameter, output <br> - Wrote a template deploying a 2-tier VPC (public + private subnets, IGW, route tables) <br> - Deployed the stack and verified the resources were created correctly <br> - Updated the stack (changed CIDR) and watched CloudFormation apply a change set | 10/06/2026 | 10/06/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 6 | - Wrote a CloudFormation template that deploys Lambda + API Gateway + DynamoDB following this week's serverless architecture <br> - Deleted the manually-created resources and re-deployed via stack to demonstrate reusability <br> - Deliberately introduced an error (S3 bucket name conflict) and watched CloudFormation **roll back** <br> - Wrap-up session with mentor and collected feedback for the final project | 12/06/2026 | 12/06/2026 | <https://cloudjourney.awsstudygroup.com/> |

### Week 8 Outcomes:

- Created and deployed Lambda functions; able to attach an **execution role** with least privilege.
- Connected API Gateway + Lambda + DynamoDB into a complete **serverless REST API** and tested it via `curl`.
- Reused **DynamoDB Streams** + Lambda + S3 from week 5 to build an event-driven pipeline.
- Wrote a **CloudFormation** template to deploy a 2-tier VPC; understood stacks, parameters and outputs.
- Observed CloudFormation **rollback** on failure, reinforcing why IaC is preferable to console clicking.
- Completed the **8-week foundation phase** and ready for the final 4 weeks focused on the project.

### Difficulties and lessons learned:

- Lambda execution role does not have CloudWatch Logs write permissions by default on a new account — I had to attach `AWSLambdaBasicExecutionRole` manually.
- API Gateway needs a **deployed stage** to expose a URL — easy to forget because the console does not warn you.
- CloudFormation rollback only applies if enabled in the stack option; otherwise already-created resources stay around and leave the stack in a half-broken state. I hit this and spent 30 minutes cleaning up manually.