# Modules

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*ItQg-iUT0O3QDiLoJBndJg.png) 

Modules are containers for multiple resources that are used together. A module consists of a collection of .tf and/or .tf.json files kept together in a directory.

Modules are the main way to package and reuse resource configurations with Terraform.

## Root module

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*ilau3dR50ZfKadoc1_TO_w.png) 

Every Terraform configuration has at least one module, known as its **root** module, which consists of the resources defined in the `.tf` files in the main working directory.

## Child Modules

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*7OER_lXpP-25B1LKW1lPOw.png) 

A Terraform module (usually the root module of a configuration) can call other modules to include their resources into the configuration. A module that has been called by another module is often referred to as a **child module**.

Child modules can be called multiple times within the same configuration, and multiple configurations can use the same child module.

## Examples

```terraform
provider "aws" {
    region = "eu-west-1"
}

module "work_queue" {
    source = "./sqs-with-backoff"
    queue_name = "work-queue" # variable
}

output "work_queue_name" {
    value = module.work_queue.queue_name # access module output
}

output "work_queue_dead_letter_name" {
    value = module.work_queue.dead_letter_queue_name # access module output
}
```

**Remote module**

```terraform
provider "aws" {
    region = "us-east-2"
}

module "work_queue" {
    source = "github.com/dinhhuy258/sqs-with-backoff"
    queue_name = "work-queue"
}

output "work_queue" {
    value = module.work_queue.queue # access module output
}

output "work_queue_dead_letter_queue" {
    value = module.work_queue.dead_letter_queue # access module output
}
```
