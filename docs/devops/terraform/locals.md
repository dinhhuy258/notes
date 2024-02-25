# Terraform Locals

Terraform Locals are named values which can be assigned and used in your code. It mainly serves the purpose of reducing duplication within the Terraform code. 

```terraform
locals {
    bucket_name = "${var.text1}-${var.text2}"
}
```

## How to Use Terraform Locals? 

- First, declare the local along and assign a value

```terraform
locals {
    env         = "dev"
    instance_ids = concat(aws_instance.ec1.*.id, aws_instance.ec3.*.id)
    prefix_elements = [for elem in ["a", "b", "c"] : format("Hello %s", elem)]
    even_numbers = [for i in [1, 2, 3, 4, 5, 6] : i if i % 2 == 0]
}
```

- Then, use the local name anywhere in the code where that value is needed

```terraform
resource "aws_s3_bucket" "my_test_bucket" {
    bucket = "${local.bucket_name}-newbucket"
    acl    = "private"
 
    tags = {
        Name        = local.bucket_name
        Environment = local.env
    }
}
```

## Terraform Locals vs Variables

**Variables**
- Terraform variable can be scoped globally
- A variable value can be manipulated via expressions
- A variable value can be set from an input or `.tfvars` file
- **Can not** use dynamic expressions

**Locals**
- A Local is only accessible within the local module
- A local in Terraform doesn’t change its value once assigned
- A local value **can not** be set from and input or `.tfvars` file
- Can use dynamic expressions

## Examples

Using locals to name resources

```terraform
locals {
    name_prefix = "JacksDemo-${var.environment}"
}

module "appgateway" {
    application_gateway_name = "${local.name_prefix}-${var.application_gateway_name}"
    ...
}
```

Using locals to set resource tags

```terraform
locals {
    mandatory_tags = {
        cost_center = var.cost_center,
        environment = var.environment
    }
}

tags = merge(var.resource_tags, local.mandatory_tags)
```
