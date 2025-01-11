# IAM

IAM is a service that controls access to AWS resources

## Components of IAM

![imgur.png](https://i.imgur.com/SQBo4HN.png)

- **IAM entities**: These are the  IAM resources to authenticate the requesting entity. These include the following:
    - IAM users
    - IAM roles

- **IAM identities**: The IAM resources that IAM uses to check the permissions scope of the requesting entity. These include the following:
    - IAM users
    - IAM roles
    - IAM groups

- **Principal:** The user, service, or application that requests access to an IAM service or a resource. It can be both an external or an internal entity.
- **Other IAM resources**: These are the IAM resources that do not fall into any of the above categories. These are used for a wide range of operations that deal with identity and access management. These include the following:
    - IAM policies
    - Identity providers
    - Access Analyzer
 
## How IAM works#

When an entity requests access to any of the AWS services or resources, that request is first analyzed by IAM. IAM checks the credentials provided by the requesting entity to authenticate it. After the requesting entity has been authenticated, it analyzes the permissions granted to the entity and checks if the current request falls into that pool of permissions

![imgur.png](https://i.imgur.com/2P4miuD.png)

## IAM policy

AM policy is a JSON document attached to the AWS resource that is used by the logged-in entity to authenticate itself or to the AWS resource to which secure access is required.

### Identity-based policies

Identity-based policies are attached to an IAM user, group, or role. These policies let you specify what that identity can do (its permissions).

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:CreateBucket"
            ],
            "Resource": [
                "arn:aws:s3:::demo-bucket"
            ]
        }
    ]
}
```

This policy allows the principal entity to create an S3 bucket by the name `demo-bucket`.

### Resource-based policies#

Resource-based policies are attached to a resource. For example, you can attach resource-based policies to Amazon S3 buckets, Amazon SQS queues... With resource-based policies, you can specify who has access to the resource and what actions they can perform on it.

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::demo-bucket/demo-object",
            "Principal": "*"
        }
    ]
}
```
This policy allows anyone to read `demo-object` stored in an S3 bucket by the name `demo-bucket`.

## Best practices

- One IAM User per person ONLY
- One IAM Role per Application
- Never use the ROOT account except for initial setup
- It's best to give users the minimal amount of permissions to perform their job
