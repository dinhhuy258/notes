# CloudWatch vs CloudTrail

- **CloudWatch** focuses on monitoring AWS services and resources' health and performance.
- **CloudTrail** logs all actions performed within your AWS environment, tracking who did what.

> CloudWatch: `What is happening on AWS?` and logging all the events for a particular service or application.
> CloudTrail: `Who did what on AWS?` and the API calls to the service or resource.

## AWS CloudWatch:

A monitoring service for AWS resources and applications.

- Collects and tracks **metrics** (e.g., CPU usage, memory, latency).
- Supports **log collection** from applications and AWS services.
- Allows setting **alarms** and triggering automated actions.
- Offers **basic monitoring (5 min)** and **detailed monitoring (1 min)**.

## AWS CloudTrail

A logging service for governance, compliance, and auditing.

- Records **API activity** across AWS accounts and services.
- Logs actions from **AWS Console, SDKs, CLI, and services**.
- Stores logs in **S3 buckets or CloudWatch Logs** for long-term retention.
- Helps track **who made changes** to AWS infrastructure.
- Free management event logging; **data event logging incurs charges**.

## Summary

- **Use CloudWatch** for monitoring application and infrastructure performance.
- **Use CloudTrail** for tracking API activity, security audits, and compliance.
- Both services are enabled by default, but detailed monitoring and additional logging may incur extra costs.

## Reference

[AWS — Difference between CloudWatch and CloudTrail](https://medium.com/awesome-cloud/aws-difference-between-cloudwatch-and-cloudtrail-16a486f8bc95)
