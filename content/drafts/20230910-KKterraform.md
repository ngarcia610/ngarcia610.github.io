---
title: "Terraform Associate"
description: "Notes from KodeKloud Terraform Associate Course"
date: 2023-09-10T18:04:05-04:00
lang: en
categories: ["Notes"]
tags: ["Terraform"]
draft: True
---

## IaC Concepts

Configuration Management Tools:

- Features:
  - Designed to install and manage software
  - Maintains standard structure
  - Version control
  - Idempotent
- Examples: Ansible, Puppet, Saltstack

Server Templating Tools

- Features:
  - Pre installed software and dependencies
  - VM or docker images
  - Immutable infrastructure
- Examples: Docker, Packer, Vagrant

Provisioning Tools

- Features:
  - Deploy immutable infrastructure resources
  - Servers, db, network components, etc.
  - Multiple providers
- Examples: Terraform, Cloudformation

Although configuration management tools like ansible can deploy infrastructure, it uses a procedural approach instead of a declarative one.

This means that if you have a script that creates 2 vms: Ansible will create 4 if run twice, while Terraform only creates 2.

Installing Terraform

```bash
wget https://releases.hashicorp.com/terraform/0.15.0/terraform_0.15.0_linux_amd64.zip
unzip terraform_0.15.0_linux_amd64.zip
mv terraform /usr/local/bin
terraform version
```

Terraform commands

- `terraform init` download provider info
- `terraform plan` review resources
- `terraform apply` review and create resources
  -auto-approve proceeds without confirmation
- `terraform show` see the details of resources created
- Terraform workflow is `init > plan > apply`

Create, Update and Destroy Infrastructure

- Resources are destroyed and recreated on changes
- `terraform destroy` removes resources

Terraform file structure

- `main.tf` main config, resource definition
- `variables.tf` variable declarations
- `outputs.tf` output from resources
- `provider.tf` provider definition
- `terraform.tf` terraform behavior

## Terraform Providers

There are two reasons to use a provider argument in the configuration:

- To override the default provider configuration.
- A configuration may need to use multiple versions of the same provider.

Helpful Commands:

- `terraform version` can show the version of the provider plugins that are downloaded in the config directory.
- `terraform providers` command displays the providers needed by the configuration.

Version constraints can be used anywhere terraform allows us to specify versions.

They can be set at:

- Provider requirements
- Modules
- The `required_version` setting in the terraform block

Example required providers block:

```HCL
terraform {
  required_providers {
    local = {
      source = "hashicorp/local"
      version = "1.4.0"
    }
  }
}
```

You can separate values with commas, and use comparison operators.

The following operators are valid:

- `=` (or no operator): Allows only one exact version number. Cannot be combined with other conditions.
- `!=` : Excludes an exact version number.
- `>, >=, <, <=`: Comparisons against a specified version, allowing versions for which the comparison is true. "Greater-than" requests newer versions, and "less-than" requests older versions.
- `~>`: Allows only the rightmost version component to increment. For example, to allow new patch releases within a specific minor release, use the full version number: ~> 1.0.4 will allow installation of 1.0.5 and 1.0.10 but not 1.1.0. This is usually called the pessimistic constraint operator.

Aliases can be used when you need to differentiate provider blocks.

In this example, there is a second provider block with the alias 'central' to differentiate it.

```HCL
provider "aws" {
  region    = "us-east-1"
}

provider "aws" {
  region    = "ca-central-1"
  alias = "central"
}
```

## Variables, Resource Attributes and Dependencies

Variables example:

```HCL
# Main.tf
resource "local_file" "pet" {
  filename = var.filename
  content = var.content
}
```

```HCL
# Variables.tf
variable "filename" {
  default = "/root/pets.txt"
}

variable "content" {
  default = "My favorite pet Taza"
}
```

You can also pass in variables on the command line with `-var` or as global variables with `export TF_VAR_<resource>`.

Terraform automatically loads variable files that:

- Are named exactly `terraform.tfvars` or `terraform.tfvars.json`.
- Any files with names ending in `.auto.tfvars` or `.auto.tfvars.json`.

Otherwise you must use `terraform apply -var-file="testing.tfvars"`

Variable Definition Precedence

1. Env variables
2. terraform.tfvars
3. `*.auto.tfvars` (alphabetical order)
4. `-var` or `-var-file` (command-line flags)

Variables can be string, number, or boolean.

Common arguments inside a variable block

- `default =` provides the default value
- `description =` what is it
- `type =` string, number, boolean
- `sensitive =` value is suppressed when running plan or apply; accepts True or False

Validation rules

- Specify whether the value must adhere to rules
- Example:

```HCL
# Forces AMI to have 'AMI-'

variable "ami" {
  type = string
  description = " the id of the machine image (AMI) to use for the server."
    validation {
      condition = substr(var.ami, 0, 4) == "ami-"
      error_message = "The AMI should start with \"ami-\"."
    }
}
```

Output variables are displayed when you run terraform apply. It is displayed even if no changes are applied.

## Terraform State

Terraform apply by default creates a state file in same directory that terraform is run.

The file is called `terraform.tfstate` and a backup file called `terraform.tfstate.backup`.

State file is in json.

When terraform plan is run, it checks to see if the state file exists, then refreshes it.

Refresh means it compares the state against real resources and changes that need to be made in the config.

You can stop the refresh behavior with `terraform apply -refresh=false` but this is not recommended.

Terraform creates resources in order based dependencies. In main.tf resource block, `depends_on =` will be specified.

Don't commit state files to remote version control systems. They can contain sensitive data as well as a feature of terraform called "state locking".

State locking maintains the integrity of the state file when multiple developers are running terraform apply.

Use remote state configurations instead.

Items needed to configure S3 as a remote state backend

- Bucket name
- key
- region
- DynamoDB table

Example of backend setup:

```HCL
# main.tf
resource "local_file" "item" {
  filename = "/root/item.txt"
  content = "some text"
}

# terraform.tf
terraform {
  backend "s3" {
    bucket = "bucket name"
    key = "key/terraform.tfstate"
    region = "us-east-1"
    dynamodb_table = "table-name"
  }
}
```

Remember to run terraform init to initialize the new backend before running apply.

Once the backend is successful, you can delete the local configuration.

## Read, Generate and Modify Configuration

Meta Arguments:

- `count` used to create a list of resources
- `for each` used to create a map of resources
- Example: When a resource is removed, a list impacts the other resources, a map impacts only the removed resource

Count and for_each:

- You want to specify multiple resources in the resource block.
- There are two options: count and for_each
- Count uses a list and names the resources the index of the list
- for_each creates resources as a map of resources
- If you remove an instance from a resource block, count and for_each behave differently:
  - count updates all resources with a new index
  - for_each updates only the resource impacted

Example of for_each:

```HCL
# main.tf
resource "aws_instance" "web" {
  ami = var.ami
  instance_type = var.instance_type
  for_each = var.webservers
  tags = {
    Name = each.value
  }
}

# variables.tf
variable "ami" {
  default = "ami-01234"
}

variable "instance_type" {
  default = "m5.large"
}

variable "webservers" {
  type = set
  default = ["web1", "web2", "web3"]
}
```

Provisioners

- Types of provisioners: remote exec, local exec, and file
- remote exec: winrm or ssh to run commands on a remote host
- local exec: commands are run on the machine that is running the tf binary

Example of a provisioner

```HCL
# main.tf
resource "aws_instance" "webserver" {
  ami = "ami-1234"
  instance_type = "t2.micro"
  provisioner "remote-exec" {
    inline = [
      "sudo apt update",
      "sudo apt install nginx -y",
      "sudo systemctl enable nginx",
      "sudo systemctl start nginx"
    ]
  }
  # Handles the authentication
  connection {
    type = "ssh"
    host = self.public_ip
    user = "ubuntu"
    private_key = file("/root/.ssh/web")
  }
  key_name = aws_key_pair.web.id
  vpc_security_group_ids = [ aws_security_group.ssh-access.id]
}

resource "aws_key_pair" "web" {
  << code hidden >>
}

```

Consequences of using provisioners:

- It adds a considerable amount of complexity and uncertainty to Terraform usage.
- Use of provisioners requires coordinating many more details than Terraform usage usually requires.
- Terraform cannot model the actions of provisioners as part of a plan.

Built-in Functions

- Terraform does not support user defined functions.
- `file()` can be used to read information from a file
- `length()` can return the length of variable
- `toset()` convert a list to a set and removes duplicate values
- `terraform console` allows you to enter the interactive prompt (allows you to experiment with functions)
- Common function types:
  - Numeric: transform and manipulate numbers
  - String: transform and manipulate strings
  - Collection: transform and manipulate sets and lists
  - Type Conversion: change between types
- Numeric Functions
  - `max()`
  - `min()`
  - `ceil()`
- String Functions
  - `split()`
  - `lower()`
  - `upper()`
  - `title()`
  - `substr()`
- Collection Functions
  - `length()`
  - `index()`
  - `element()`
  - `contains()`
- Map Functions (no longer available)
  - `keys()`
  - `values()`
  - `lookup()`

Conditional Expressions

- Format of a conditional `condition ? true_val : false_val`
- Example: `length = var.length < 8 ? 8 : var.length`
  - The condition checks if the variable is less than 8
  - If true, use '8' as the value of the length argument
  - Else, make use of the length variable as the value of length

Local Values

- 

Dynamic Blocks and Splat Expressions

- 

## Terraform CLI

Lifecycle rules

- `prevent_destroy = true` rejects any changes resulting in the resource being destroyed
- `ignore_changes = true` reverts changes in the config, or directly to the resource, to the original value; accepts a list
- `create_before_destroy = true` the new replacement object is created first, and the prior object is destroyed after the replacement is created

Terraform Taint

- If a resource fails to be created for any reason, it will be marked as tainted.
- You can taint or untaint resources with the respective command, however these commands are now legacy.
- Use `terraform apply -replace="<resource_name>"` as of v1.5.x

Debugging

- Set the variable `TF_LOG=<log_value>`
- There are 5 log values (levels) that can be selected
- From least to most verbose: Info, Warning, Error, Debug, Trace
- You can edit the log location with the `TF_LOG_PATH=/tmp/terraform.log` variable
- Disable logging with `unset TF_LOG_PATH`

Terraform Import

- You can import existing resources into terraform using `terraform import`
- You need a unique attribute to import (such as instance_id)
- The syntax is `terraform import <options> <address> <id>`
- This command only adds the resource to the state file, you must manually add them to the config.
- `terraform plan` can then be used to refresh the state
- You can also import local resources using `data sources`

Terraform Workspace

- Create environments without duplicating config files
- Subcommands include: show, new, list, select, and delete
- `terraform workspace list` shows existing workspaces
- '\*' shows the current workspace
- `terraform workspace new production` creates a new workspace called 'production'
- This is useful for creating dev and production workspaces.
- You can edit your variables to have different values for dev and prod:

```HCL
variable "instance_type" {
  type = map
  default = {
    "development" = "t2.micro"
    "production" = "m5.large"
  }
}
```

- You can then use the `lookup()` function to return the value based on the workspace.
- Switch workspaces using `terraform workspace select <workspace_name>`
- Include the name of the workspace using `${terraform.workspace}`
- Workspace states are stored in `terraform.tfstate.d`

## Terraform Modules

- 

## Terraform Cloud

- 
