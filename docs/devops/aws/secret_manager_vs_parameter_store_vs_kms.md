# AWS Secrets Manager vs Parameter Store vs KMS

**AWS Secrets Manager**: Secure storage, management, and automatic rotation of secrets like API keys and database credentials.
**AWS Parameter Store**: Centralized storage for configuration values, including both sensitive and non-sensitive data.
**AWS KMS (Key Management Service)**: Manages encryption keys used for securing data but does not store secrets.

## Similarities

**Encryption** – All three support encryption via **AWS KMS** for secure data storage.
**IAM Integration** – Fine-grained access control through IAM policies.
**Auditing & Logging** – Integration with AWS CloudTrail for tracking access and usage.

## Key Differences

| Feature                  | AWS Secrets Manager                                                            | AWS Parameter Store                                        | AWS KMS                                      |
| ------------------------ | ------------------------------------------------------------------------------ | ---------------------------------------------------------- | -------------------------------------------- |
| **Primary Use Case**     | Storing and managing secrets (API keys, database credentials, passwords, etc.) | Storing application configurations and secrets             | Managing encryption keys for securing data   |
| **Storage**              | Stores secrets with encryption                                                 | Stores parameters (both encrypted and unencrypted)         | Does not store secrets, only encryption keys |
| **Secret Rotation**      | Built-in support for automatic rotation                                        | Manual rotation using AWS Lambda                           | Not applicable                               |
| **Cost**                 | $0.40/month per secret + API call costs                                        | Free for standard parameters; paid for advanced parameters | $1/month per key + API call costs            |
| **Cross-Account Access** | Supported                                                                      | Not supported                                              | Supported                                    |
| **Multi-Region Support** | Supports replication across AWS regions                                        | Not supported                                              | Supports multi-region keys                   |
| **Versioning**           | Supports multiple active versions                                              | Supports versioning but only one active version            | Not applicable                               |
| **Limits**               | 500,000 secrets per region                                                     | 10,000 standard parameters per region                      | No strict limit on key storage               |

## When to Choose What?

- **Use AWS Secrets Manager** if you need **secure storage, encryption, and automatic secret rotation** (e.g., for database credentials, API keys).
- **Use AWS Parameter Store** if you need a **cost-effective way to store both configuration settings and secrets**.
- **Use AWS KMS** if you need **encryption key management** to secure data across AWS services.

## Reference

[AWS — Difference between Secrets Manager and Parameter Store (Systems Manager)](https://medium.com/awesome-cloud/aws-difference-between-secrets-manager-and-parameter-store-systems-manager-f02686604eae)
