# App Config

AppConfig is a service offered by AWS for the purpose of centralizing management of configuration data. It gives you the ability to create, manage, and deploy configuration changes separate from code

## Why use AWS AppConfig

- **Safe Deployments**: Monitor successful deployments with configurable strategies (linear, canary, all-at-once)
- **Automatic Rollback**: Built-in rollback capabilities if deployment fails or errors exceed thresholds
- **Configuration Validation**: Validate configurations using JSON Schema or Lambda functions before deployment
- **Feature Flags**: Quickly enable/disable features in production without code deployments
- **Dynamic Configuration**: Adjust application behavior in real-time without redeployment
- **Improved Resiliency**: Reduce risks associated with configuration changes through progressive rollouts

## Comparison between Parameter Store vs AppConfig

| Feature                | Parameter Store              | AppConfig                                  |
| ---------------------- | ---------------------------- | ------------------------------------------ |
| **Purpose**            | Simple key-value storage     | Advanced configuration management          |
| **Deployment Control** | No deployment management     | Progressive rollouts (linear, canary)      |
| **Validation**         | None                         | JSON Schema or Lambda validators           |
| **Rollback**           | Manual                       | Automatic on failure                       |
| **Monitoring**         | Basic                        | Comprehensive deployment monitoring        |
| **Best For**           | Static configuration values  | Dynamic configurations, feature flags      |
| **Use Case**           | Simple configuration storage | Production config changes requiring safety |

**AWS Recommendation**: Use Secrets Manager for secrets, Parameter Store for simple key-value pairs, and AppConfig for feature flags and advanced dynamic configuration.

## Configure AppConfig via Terraform

### Required Setup Order

1. Create an Application
2. Create an Environment
3. Create a Configuration Profile
4. Create a Deployment Strategy
5. Deploy the Configuration

### Basic Example

```hcl
# 1. Create AppConfig Application
resource "aws_appconfig_application" "example" {
  name        = "my-application"
  description = "My application configuration"

  tags = {
    Environment = "production"
  }
}

# 2. Create Environment
resource "aws_appconfig_environment" "example" {
  name           = "production"
  description    = "Production environment"
  application_id = aws_appconfig_application.example.id
}

# 3. Create Configuration Profile
resource "aws_appconfig_configuration_profile" "example" {
  application_id = aws_appconfig_application.example.id
  name           = "my-config-profile"
  description    = "Configuration profile"
  location_uri   = "hosted"

  validator {
    type    = "JSON_SCHEMA"
    content = jsonencode({
      "$schema" = "http://json-schema.org/draft-07/schema#"
      type      = "object"
      properties = {
        featureEnabled = { type = "boolean" }
      }
    })
  }
}

# 4. Create Deployment Strategy
resource "aws_appconfig_deployment_strategy" "example" {
  name                           = "my-deployment-strategy"
  description                    = "Linear deployment over 10 minutes"
  deployment_duration_in_minutes = 10
  growth_factor                  = 10
  replicate_to                   = "NONE"
}
```

### Using Terraform Module

```hcl
module "appconfig" {
  source  = "terraform-aws-modules/appconfig/aws"
  version = "~> 2.0"

  name        = "my-application"
  description = "My application configuration"

  environments = {
    nonprod = { name = "nonprod" }
    prod    = { name = "production" }
  }

  hosted_configuration_versions = {
    feature_flags = {
      application_id           = module.appconfig.application_id
      configuration_profile_id = module.appconfig.configuration_profile_id
      content                  = jsonencode({ featureEnabled = true })
      content_type            = "application/json"
    }
  }
}
```
