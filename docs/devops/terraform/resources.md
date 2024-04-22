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
resource "aws_vpc" "my_vpc" {
    cidr_block = "10.0.0.0/16"
}

resource "aws_security_group" "my_security_group" {
    vpc_id = aws_vpc.my_vpc.id
    name = "Example security group"
}

resource "aws_security_group_rule" "tls_in" {
    protocol = "tcp"
    security_group_id = aws_security_group.my_security_group.id
    from_port = 443
    to_port = 443
    type = "ingress"
    cidr_blocks = ["0.0.0.0/0"]
}
```

## Resource Types

Each resource is associated with a single resource type, which determines the kind of infrastructure object it manages and what arguments and other attributes the resource supports.

## References

- [https://developer.hashicorp.com/terraform/language/resources](https://developer.hashicorp.com/terraform/language/resources)
