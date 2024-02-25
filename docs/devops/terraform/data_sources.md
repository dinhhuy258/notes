# Data Sources

Data sources allow Terraform to use information defined outside of Terraform, defined by another separate Terraform configuration, or modified by functions.

## Using Data Sources

```terraform
data "aws_ami" "example" {
  most_recent = true

  owners = ["self"]
  tags = {
    Name   = "app-server"
    Tested = "true"
  }
}
```

A data block requests that Terraform read from a given data source (`aws_ami`) and export the result under the given local name (`example`). The name is used to refer to this resource from elsewhere in the same Terraform module, but has no significance outside of the scope of a module.

The data source and name together serve as an identifier for a given resource and so must be unique within a module.

Within the block body (between `{` and `}`) are query constraints defined by the data source. Most arguments in this section depend on the data source, and indeed in this example `most_recent`, `owners` and `tags` are all arguments defined specifically for the `aws_ami` data source.

## Examples

**Using data sources to access external resource attributes**

```terraform
data "aws_s3_bucket" "existing_bucket" {
  bucket = "sumeet.life"
}
```

I have a bucket named `sumeet.life` in my AWS account.

When this Terraform configuration with appropriate provider settings is initialized and enabled, Terraform reads the information from AWS and makes it available in the `data.aws_s3_bucket.existing_bucket` variable.

**Managing resource dependencies with data sources**

Data sources indirectly help manage resource dependencies. If the data being queried by data sources does not exist, then the resource that is dependent on the same will not be created.

```terraform
data "aws_subnets" "my_subnets" {
  filter {
    name   = "vpc-id"
    values = ["vpc-xxx"]
  }
}

resource "aws_instance" "my_vm_2" {
  for_each      = toset(data.aws_subnets.my_subnets.ids)
  ami           = var.ami //Ubuntu AMI
  instance_type = var.instance_type

  subnet_id = each.key

  tags = {
    Name = var.name_tag,
  }

  depends_on = [ data.aws_subnets.my_subnets ]
}
```

**Validate inputs with data sources**

```terraform
variable "instance_ami" {
  description = "Ubuntu AMI in Frankfurt region."
  //default = "ami-065deacbcaac64cf2"
  default = "ami-xxx" // This does not exist
}


resource "aws_instance" "myec2" {
  ami = var.instance_ami
  instance_type = "t2.micro"
}
```

If we provide invalid `instance_ami` the terraform output does not throw any error

```bash
.
.
.
+ user_data                            = (known after apply)
      + user_data_base64                     = (known after apply)
      + user_data_replace_on_change          = false
      + vpc_security_group_ids               = (known after apply)
    }

Plan: 1 to add, 0 to change, 0 to destroy.
```

However, it does throw an error when we proceed to apply this configuration since the given AMI does not exist.

In certain scenarios, it might be desirable for this configuration to throw an error in the planning phase itself for various reasons. This is achieved using data sources.

```terraform
data "aws_ami" "selected" {
  most_recent = true
  filter {
    name = "image-id"
    values = [var.instance_ami]
  }
}
```

If we provide invalid `instance_ami` the terraform plan output 

```bash
.
.
.
+ user_data_base64                     = (known after apply)
      + user_data_replace_on_change          = false
      + vpc_security_group_ids               = (known after apply)
    }

Plan: 1 to add, 0 to change, 0 to destroy.
╷
│ Error: Your query returned no results. Please change your search criteria and try again.
│ 
│   with data.aws_ami.selected,
│   on main.tf line 9, in data "aws_ami" "selected":
│    9: data "aws_ami" "selected" {
```

**Using data sources to access AWS secrets**

Secrets Manager is a service provided by AWS to manage sensitive data. It stores the sensitive variables securely using encryption, and makes them available to various services for automation purposes.

In Terraform configurations, these secrets are accessed using data sources.

```terraform
data "aws_secretsmanager_secret" "mydb_secret" {
  arn = "arn:aws:secretsmanager:eu-central-1:532199187081:secret:pg_db-dKuRrd"
}

data "aws_secretsmanager_secret_version" "mydb_secret_version" {
  secret_id = data.aws_secretsmanager_secret.mydb_secret.id
}

resource "aws_db_instance" "my_db" {
  allocated_storage = 20
  storage_type = "gp2"
  engine = "postgres"
  engine_version = "13.3"
  instance_class = "db.t2.micro"
  username = jsondecode(data.aws_secretsmanager_secret_version.mydb_secret_version.secret_string).username
  password = jsondecode(data.aws_secretsmanager_secret_version.mydb_secret_version.secret_string).password
  skip_final_snapshot = true


  tags = {
    Name = "ExampleDB"
  }

  tags_all = {
    Environment = "Development"
  }
}
```
