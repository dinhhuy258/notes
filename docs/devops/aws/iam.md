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
 
## How IAM works

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

### Resource-based policies

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


## AWS IAM Roles

![imgur.png](https://i.imgur.com/FeBCqE4.png)

AWS IAM Roles are a feature of IAM that allows you to grant permissions to entities that you trust to access your AWS resources. An entity can be an AWS service, an IAM user, an IAM role, or an external identity provider. By using IAM roles, you can delegate access to your AWS resources without sharing your long-term credentials, such as your IAM username and password or access keys.

### How IAM Roles Work

An IAM role is an IAM identity that has a set of permissions attached to it. Unlike an IAM user, an IAM role does not have its own credentials. Instead, an entity that assumes an IAM role is temporarily granted the permissions of the role. The entity can then use those permissions to access your AWS resources.

To assume an IAM role, an entity needs to have permission to do so. This permission is granted by a trust policy that is attached to the role. The trust policy defines which entities are allowed to assume the role. For example, you can create a trust policy that allows an EC2 instance to assume a role, or a trust policy that allows a user from another AWS account to assume a role.

When an entity assumes an IAM role, it receives a set of temporary security credentials that consist of an access key ID, a secret access key, and a session token. The entity can use these credentials to make API requests to AWS services. The credentials are valid for a limited time, typically from a few minutes to several hours. When the credentials expire, the entity can request new credentials or stop using the role.

![imgur.png](https://i.imgur.com/bsjvXuH.png)

Example of trust policy 

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "sts:AssumeRole"
            ],
            "Principal": {
                "Service": [
                    "ec2.amazonaws.com"
                ]
            }
        }
    ]
}
```

## Best practices

- One IAM User per person ONLY
- One IAM Role per Application
- Never use the ROOT account except for initial setup
- It's best to give users the minimal amount of permissions to perform their job
