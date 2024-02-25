# Terraform

## Terraform Lifecycle

Terraform lifecycle consists of – init, plan, apply, and destroy.

1. **Terraform init** initializes the (local) Terraform environment. Usually executed only once per session.
2. **Terraform plan** compares the Terraform state with the as-is state in the cloud, build and display an execution plan. This does not change the deployment (read-only).
3. **Terraform apply** executes the plan. This potentially changes the deployment.
4. **Terraform destroy** deletes all resources that are governed by this specific terraform environment.

## Terraform Providers

A provider is responsible for understanding API interactions and exposing resources. It is an executable plug-in that contains code necessary to interact with the API of the service. Terraform configurations must declare which providers they require so that Terraform can install and use them.

![](https://k21academy.com/wp-content/uploads/2020/11/Terraform-provider-api-call.png) 

## Terraform Configuration Files

![](https://k21academy.com/wp-content/uploads/2020/11/terraform-config-files-e1605834689106.png) 

1. Configuration file (*.tf files): Here we declare the provider and resources to be deployed along with the type of resource and all resources specific settings
2. Variable declaration file (variables.tf or variables.tf.json): Here we declare the input variables required to provision resources
3. Variable definition files (terraform.tfvars): Here we assign values to the input variables
4. State file (terraform.tfstate): a state file is created once after Terraform is run. It stores state about our managed infrastructure.
