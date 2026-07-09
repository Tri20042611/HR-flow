---
title: "Blog 2"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.2. </b> "
---

# SAM – Deploy AWS Without Clicking

In my previous post, I shared about Kiro. Thanks for all the interest! Today I'm continuing with something that has saved me a ton of frustration: SAM. For me, this is truly a "whole new world" as a complete newbie.

## What is SAM?

Simply put, SAM (Serverless Application Model) is a tool for writing Infrastructure as Code for AWS serverless services like Lambda, API Gateway, S3, DynamoDB, SQS, EventBridge... Think of it like Terraform, but built by AWS specifically for their ecosystem.

For the full details:
https://docs.aws.amazon.com/serverless-application-model/

When I first started learning AWS, before I knew about Kiro or SAM, I deployed everything through the AWS Console.

- Click to create Lambda.
- Click to create S3.
- Click to attach trigger.
- Click to edit IAM.
- Click to deploy.

A few times is fine, but when a project has many services, it starts to feel like a "time killer." There were days when I only needed to edit one Lambda but spent almost ten minutes clicking through everything, and forgetting one step meant starting over.

At that time, I was also working on a small project. Looking back, the code was quite rough, so I won't go into detail, but the time spent on deployment actually exceeded the time spent on writing code.

Then I discovered Kiro and SAM. Like adding wings to a tiger.

## What I love most about SAM: the workflow

Here's my current workflow:

1. Write `template.yaml` to declare the entire infrastructure: Lambda, S3, DynamoDB, SQS, EventBridge...
2. `sam build` — SAM transforms the template into CloudFormation.
3. `sam deploy` — deploy to AWS, with a Change Set to review what will change before applying.

![SAM Workflow](/images/3-BlogsPosted/3.2-Blog2/sam_workflow.png)

The feeling of going from clicking through each service to just editing a YAML file and running a few commands.

## The second thing I really like: SAM Policies

Instead of writing a long IAM Policy JSON like this:

```json
{
  "Effect": "Allow",
  "Action": [
    "s3:PutObject",
    "s3:GetObject"
  ],
  "Resource": "arn:aws:s3:::my-bucket/*"
}
```

Now it's just:

```yaml
Policies:
  - S3CrudPolicy:
      BucketName: !Ref MyBucket
```

One look and you get it — concise and no need to memorize all the ARN formats.

## The "headaches" when using SAM

### Circular Dependency

This was probably the first wall I hit.

I spent an entire afternoon trying to understand what was going on (actually, AI explained it to me before I got it 😅).

My case:

- S3 Bucket needs to trigger Lambda when a file is uploaded.
- But Lambda also references that same Bucket to read the file.
- CloudFormation looks at this and doesn't know whether to create the Bucket first or the Lambda first.

So it reports:

> "Circular dependency between resources."

When I first saw the error, I was like: "Wait, how is this related?"

Once I understood the cause, I realized CloudFormation actually has a point.

### Vibe Coding made me... lose my mind

With SAM + AI Agent, it's truly a joy.

I just describe the requirements, and AI writes most of it.

But because of that, sometimes I look back at my own project and have to read through everything from scratch to understand how it's running.

Saving time? Yes.

But overusing it is also a bit concerning.

## Who should use SAM, in my opinion?

- Building backend on AWS.
- Using many Lambdas or many serverless services.
- Already using Kiro or AI Agents for coding — SAM is practically made for AI to read and modify.

I'm still a complete AWS newbie, so there's definitely a lot I don't know yet. If any of you have experience with SAM, CloudFormation, or Serverless, I'd really appreciate your feedback so I can learn more.
