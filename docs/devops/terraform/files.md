# Files

## Terraform files

**policy.iam**

```terraform
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
            "s3:ListBucket"
            ],
            "Effect": "Allow",
            "Resource": "*"
        }
    ]
}
```

**main.tf**

```terraform
provider "aws" {
    region = "eu-west-1"
}

resource "aws_iam_policy" "my_bucket_policy" {
    name = "list-buckets-policy"
    policy = file("./policy.iam")
}

```

## templatefile

Sometimes we want to use a file but do not know all of the values before we run the project. Other times, the values are dynamic and generated as a result of a created resource. To use dynamic values in a file, we need to use the `templatefile` function.

The `templatefile` function allows us to define placeholders in a template file and then pass their values at runtime.

**example.tpl**

```
hello there ${name}
there are ${number} things to say
```

**main.tf**

```terraform


locals {
    rendered = templatefile("./example.tpl", { name = "kevin", number = 7})
}

output "rendered_template" {
    value = local.rendered
}
```

Project output:

```bash
Apply complete! Resources: 0 added, 0 changed, 0 destroyed.

Outputs:

rendered_template =
hello there kevin
there are 7 things to say
```

### Loops in a Template

You can pass in an array of values into a template and loop through them.

**backends.tpl**

```
%{ for addr in ip_addrs ~}
backend ${addr}:${port}
%{ endfor ~}
```

**main.tf**

```terraform
output "rendered_template" {
    value = templatefile("./backends.tpl", { port = 8080, ip_addrs = ["10.0.0.1", "10.0.0.2"] })
}
```

Project output:

```bash
backend 10.0.0.1:8080
backend 10.0.0.2:8080
```
