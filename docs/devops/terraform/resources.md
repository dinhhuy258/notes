# Terraform Resources

Terraform resources are components within a Terraform configuration that represent infrastructure objects or services that need to be managed.

## Terraform resource syntax

```terraform
resource "resource_type" "resource_name" {
  # Configuration settings for the resource
  attribute1 = value1
  attribute2 = value2
  # ...
}
```

For example:

```terraform
resource "aws_instance" "web" {
  ami           = "ami-a1b2c3d4"
  instance_type = "t2.micro"
}
```

## Resource Types

Each resource is associated with a single resource type, which determines the kind of infrastructure object it manages and what arguments and other attributes the resource supports.

### Providers

A provider is a plugin for Terraform that offers a collection of resource types. Each resource type is implemented by a provider.

```terraform
# https://developer.hashicorp.com/terraform/language/providers/requirements
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.22"
    }
  }
}

# provider configuration
# https://developer.hashicorp.com/terraform/language/providers/configuration 
provider "aws" {
  region = "ap-southeast-2"
}
```

List of supported providers can be found [here](https://registry.terraform.io/browse/providers)

## References

- [https://developer.hashicorp.com/terraform/language/resources](https://developer.hashicorp.com/terraform/language/resources)
