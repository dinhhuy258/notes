# Input Variables

Terraform input variables are used as parameters to input values at run time to customize our deployments. Input terraform variables can be defined in the `main.tf` configuration file but it is a best practice to define them in a separate `variable.tf` file to provide better readability and organization.

A variable is defined by using a variable block with a label. The label is the name of the variable and must be unique among all the variables in the same configuration.

The variable declaration can optionally include three arguments:

- description: briefly explain the purpose of the variable and what kind of value is expected.
- type: specifies the type of value such as string, number, bool, map, list, etc.
- default: If present, the variable is considered to be optional and if no value is set, the default value is used.

## Input Variable Types

The type argument in a variable block allows you to enforce [type constraints](https://developer.hashicorp.com/terraform/language/expressions/type-constraints) on the variables a user passes in.
Terraform supports a number of types, including string, number, bool, list, map, set, object, tuple, and any. If a type isn’t specified, then Terraform assumes the type is any.

```terraform
variable "vpcname" {
    type = string
    default = "myvpc"
}

variable "mylist" {
    type = list(string)
    default = ["value1", "value2"]
}

variable "mymap" {
    type = map
    default = {
        key1 = "value1"
        key2 = "value2"
    }
}

variable "myobject" {
    type = object({
        field1: string
        field2: string
    })
}
```

## Use Input Variables

After declaration, we can use the variables in our `main.tf` file by calling that variable in the `var.<name>` format. The value to the variables is assigned during terraform apply.

```terraform
provider "azurerm" {
    version = "1.38.0"
}

resource "azure_resource_group" "TFAzure-rg" {
    name = var.resourceGroupName
    location = var.location
    tag = {
        Environment = "Dev"
    }
}
```

## Assign Values To Input Variables

### Command-line flags

```bash
terraform apply -var="resourceGroupName=terraformdemo-rg" -var="location=eastus"
```

### Variable Definition (.tfvars) Files

```terraform
resourceGroupName = "terraformdemo-rg"
location = "eastus"
```

If there are many variable values to input, we can define them in a variable definition file. Terraform also automatically loads a number of variable definitions files if they are present:
- Files named exactly `terraform.tfvars` or `terraform.tfvars.json`
- Any files with names ending in `.auto.tfvars` or `.auto.tfvars.json`

If the file is named something else, then use the `-var-file` flag directly to specify a file.

```bash
terraform apply -var-file="testing.tfvars"
```
