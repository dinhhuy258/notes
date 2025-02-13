# Lambda

AWS Lambda is a serverless compute service that allows us to run code without provisioning or managing servers.

## Cold starts in Lambda function

One of the reasons AWS Lambda is cost-effective is that you only pay when your code runs, unlike EC2 servers that incur costs even when idle. However, the trade-off is that when a Lambda function is invoked after a period of inactivity, it takes some time to initialize the environment before running your code. This delay is known as a `cold start`.

AWS keeps Lambda environments `warm` for a short period after execution to allow subsequent invocations without a `cold start`. `Cold starts` occur roughly 1% of the time under typical conditions, but spiky or unpredictable loads can lead to more frequent cold starts.

**Stages of Lambda Invocation**

1. Download Code: Code is loaded from S3 or ECR.
2. Start Execution Environment: A new container is initialized.
3. Execute Initialization Code: Global objects or connections are initialized.
4. Execute Handler Code: The core function logic is run.

The first two stages are responsible for cold starts and are not charged. However, the third step contributes to cold start latency from a user perspective. If the same Lambda environment is reused, only the handler code (step 4) is executed.

**Persistent Resources**

Global variables, such as database connections, may persist between Lambda invocations because the environment can be reused.

**Warmers and Scalability Considerations**

Using a `warmer` function to invoke the Lambda every minute can help reduce cold starts, but this strategy is only effective for single-threaded scenarios. When multiple simultaneous requests are received, new Lambda environments are spun up, each experiencing its own cold start.

AWS Lambda may invoke new environments across different availability zones to balance load, so quick successive requests may not reuse the same environment.

**Cold Start Duration**

Cold starts generally last between 100 ms and 1 second. Although you are not charged for environment setup, the additional latency can impact the user experience.

## Lambda Layers

Lambda layers allow us to centrally manage and share code, libraries, and dependencies across multiple Lambda functions. Instead of including all dependencies within the deployment package of each function, Lambda layers enable us to package common components separately in a .zip file, making it easier to maintain, update, and reuse code.

![imgur.png](https://i.imgur.com/KZmviNl.png)

## Lambda limits

The Lambda function has certain limitations as compared to other computing services

- The Lambda function's maximum execution time can be increased to 900 seconds (15 minutes). We would have to switch to other computing options for workloads with execution time of more than 15 minutes.
- The maximum size of the compressed deployment package for the Lambda function is 50 MB. On the other hand size of uncompressed deployments, including code and dependencies, can exceed to 250 MB.
- The disk capacity in the function container ranges from 512 MB to 10 GB.
- By default, Lambda supports 1000 concurrent executions in an account. However, this number can be increased.
- The maximum size of environment variables of a function is 4 KB.

## Good to read

[Lambda Cold Starts and Bootstrap Code](https://lucvandonkersgoed.com/2022/04/08/lambda-cold-starts-and-bootstrap-code/)
