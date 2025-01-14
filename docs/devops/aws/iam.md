# IAM

IAM is a service that controls access to AWS resources

## Components of IAM

![imgur.png](https://i.imgur.com/SQBo4HN.png)

- **IAM entities**: These are the IAM resources to authenticate the requesting entity. These include the following:

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

**The structure of Identity-based policy**

| Column1   | Explanation                                                                                                |
| --------- | ---------------------------------------------------------------------------------------------------------- |
| Effect    | This specifies if the associated actions are allowed or denied. Its value can either be `Allow` or `Deny`. |
| Action    | This contains a list of actions.                                                                           |
| Resource  | This contains resources on which the actions will be effective.                                            |
| Condition | This contains resources on which the actions will be effective.                                            |

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:CreateBucket"],
      "Resource": ["arn:aws:s3:::demo-bucket"]
    }
  ]
}
```

This policy allows the principal entity to create an S3 bucket by the name `demo-bucket`.

### Resource-based policies

Resource-based policies are attached to a resource. For example, you can attach resource-based policies to Amazon S3 buckets, Amazon SQS queues... With resource-based policies, you can specify who has access to the resource and what actions they can perform on it.

**The structure of Resource-based policy**

| Column1   | Explanation                                                                                                              |
| --------- | ------------------------------------------------------------------------------------------------------------------------ |
| Effect    | This specifies if the associated actions are allowed or denied. Its value can either be `Allow` or `Deny`.               |
| Action    | This contains a list of actions.                                                                                         |
| Resource  | This contains resources on which the actions will be effective.                                                          |
| Principal | This specifies the principal (such as an IAM user, roles, and others) who is allowed (or denied) access to the resource. |
| Condition | This contains resources on which the actions will be effective.                                                          |

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

![imgur.png](https://i.imgur.com/4hFLLFx.png)

AWS IAM Roles are a feature of IAM that allows you to grant permissions to entities that you trust to access your AWS resources. An entity can be an AWS service, an IAM user, an IAM role, or an external identity provider. By using IAM roles, you can delegate access to your AWS resources without sharing your long-term credentials, such as your IAM username and password or access keys.

### How IAM Roles Work

An IAM role is an IAM identity that has a set of permissions attached to it. Unlike an IAM user, an IAM role does not have its own credentials. Instead, an entity that assumes an IAM role is temporarily granted the permissions of the role. The entity can then use those permissions to access your AWS resources.

To assume an IAM role, an entity needs to have permission to do so. This permission is granted by a trust policy that is attached to the role. The trust policy defines which entities are allowed to assume the role. For example, you can create a trust policy that allows an EC2 instance to assume a role, or a trust policy that allows a user from another AWS account to assume a role.

When an entity assumes an IAM role, it receives a set of temporary security credentials that consist of an access key ID, a secret access key, and a session token. The entity can use these credentials to make API requests to AWS services. The credentials are valid for a limited time, typically from a few minutes to several hours. When the credentials expire, the entity can request new credentials or stop using the role.

![imgur.png](https://i.imgur.com/lhlBY5d.png)

Example of trust policy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["sts:AssumeRole"],
      "Principal": {
        "Service": ["ec2.amazonaws.com"]
      }
    }
  ]
}
```

## Restricting Policies

### Permission boundary

AWS supports permissions boundaries for IAM entities (users or roles). A permissions boundary is an advanced feature for using a managed policy to set the maximum permissions that an identity-based policy can grant to an IAM entity. An entity's permissions boundary allows it to perform only the actions that are allowed by both its identity-based policies and its permissions boundaries.

![imgur.png](https://i.imgur.com/YZYfwyU.png)

### Session policy

Similar to permission boundaries, a session policy is a restricting policy. Its application is however different from that of permission boundary. Session policies are used to set an upper bound on the permissions of a session which is created when a **role is assumed**. It can only be attached when a role is assumed programatically.

**When do we need a session policy?**

An IAM role can be used to create multiple sessions at once. By default, the permissions of each session are the ones allowed by the IAM policy attached with the role. Consider a scenario where we've created a role to provide temporary access to external individuals in different roles. We have an admin access policy attached with the role that allows the assuming entities to perform all the actions within that account. What we want next is a tool to limit the permissions of each session based on the role of the individuals assuming the role. This is where session policy comes in. We can attach a session policy with each session, based on the role of the individual allowing them to perform only the actions that should be allowed to that entity.

So if we're creating a session for an entity to manage DynamoDB operations, we'll attach a session policy that'll restrict that entity from doing anything out of the scope of their role.

![imgur.png](https://i.imgur.com/5SB2h2Z.png)

## Permission intersections

### Permissions boundary with identity-based policy

**Permissions boundary as a superset of identity-based policy**: Only the permissions that are common in both the identity-based policy and permissions boundary are effective. Any additional permissions in the permissions boundary are ineffective.

![imgur.png](https://i.imgur.com/MKjzr7v.png)

**Permissions boundary as a subset of identity-based policy**: Only the permissions that are common in both the identity-based policy and permissions boundary are effective. Any additional permissions in the identity-based policy are ineffective

![imgur.png](https://i.imgur.com/iJYE4Bt.png)

**Overlapping permissions boundary and identity-based policy**: Only the permissions that are common in both the identity-based policy and permissions boundary are effective. Any additional permissions in the identity-based policy or permissions boundary are ineffective.

![imgur.png](https://i.imgur.com/DeLX4uL.png)

**Denial effect in permissions boundary or identity-based policy**: The deny permissions defined in the permissions boundary always take precedence. However, other than that, only the common permissions in the identity-based policy and permissions boundary are effective.

**_NOTE:_** The denial effect in any policy always takes precedence, even if the action is common in all the relevant policies.

**Permissions boundary, identity-based policy and resource-based policy**: The deny action also takes precedence in this case. Apart from that, the common permissions in both the identity-based policy and permissions boundary are allowed, **AND** all the permissions mentioned in the resource-based policy are allowed.

![imgur.png](https://i.imgur.com/CLqVYMi.png)

## Best practices

- One IAM User per person ONLY
- One IAM Role per Application
- Never use the ROOT account except for initial setup
- It's best to give users the minimal amount of permissions to perform their job
