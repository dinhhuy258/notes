# Terraform

## Terraform Lifecycle

Terraform lifecycle consists of – init, plan, apply, and destroy.

1. **Terraform init** initializes the (local) Terraform environment. Usually executed only once per session.
2. **Terraform plan** compares the Terraform state with the as-is state in the cloud, build and display an execution plan. This does not change the deployment (read-only).
3. **Terraform apply** executes the plan. This potentially changes the deployment.
4. **Terraform destroy** deletes all resources that are governed by this specific terraform environment.

## Terraform Configuration Files

![](https://k21academy.com/wp-content/uploads/2020/11/terraform-config-files-e1605834689106.png) 

1. Configuration file (*.tf files): Here we declare the provider and resources to be deployed along with the type of resource and all resources specific settings
2. Variable declaration file (variables.tf or variables.tf.json): Here we declare the input variables required to provision resources
3. Variable definition files (terraform.tfvars): Here we assign values to the input variables
4. State file (terraform.tfstate): a state file is created once after Terraform is run. It stores state about our managed infrastructure.

## Terraform Providers

A provider is responsible for understanding API interactions and exposing resources. It is an executable plug-in that contains code necessary to interact with the API of the service. Terraform configurations must declare which providers they require so that Terraform can install and use them.

![](https://k21academy.com/wp-content/uploads/2020/11/Terraform-provider-api-call.png) 

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

In the Terraform block above, we have defined a `required_providers` block. This required provider block allows us to specify extra properties for each of the providers we are using in the project. Under required providers, we open an AWS block. There, we specify a version constraint for the AWS provider.

List of supported providers can be found [here](https://registry.terraform.io/browse/providers)

## Backend Configuration

A backend defines where Terraform stores its state data files. Terraform uses persisted state data to keep track of the resources it manages. 

By default, Terraform uses a backend called `local`, which stores state as a local file on disk. You can also configure one of the built-in backends supported by terraform.

```terraform
terraform {
  backend "s3" {
    bucket = "mybucket"
    key    = "path/to/my/key"
    region = "us-east-1"
  }
}
```
