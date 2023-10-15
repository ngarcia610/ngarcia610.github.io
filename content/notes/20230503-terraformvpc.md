---
title: "Using Terraform to create an AWS Virtual Private Network"
date: 2023-05-03T16:47:15-04:00
categories: ["How-To"]
tags: ["Terraform"]
---

In this article, we'll see how to setup Terraform and deploy a simple AWS network using infrastructure as code (IaC).

## Introduction and Topology

Although it wouldn't take long to deploy this using the AWS console, it quickly becomes repetitive the more you have to configure.

We'll be using the us-east-1 region to create a new vpc with 3 public and private subnets across three availability zones. We'll create a public and private route table and a NAT gateway for the public subnet.

## Terraform Setup

First we setup a new account in the AWS console with permissions to deploy the infrastructure. Then we'll configure our local environment to run Terraform.

AWS Setup

Note: This example is not using best security practices because I'm going to immediately delete the infrastructure and associated user account after this lab.

1. Login to aws.amazon.com (I used the root account)
2. Go to "IAM" > Users > Add new user
3. On "Specify user Details" Create a new user named "terraform"
4. Click next to set permissions (I used "Administrator" for the example)
5. Click next to Review and Create, no additional info is needed.
6. Once the account is created click on the username link, then Security credentials tab
7. Under "Access keys" create a new access key.
8. Save the information for AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY, you'll need to paste that information later during deployment.

Note: Always protect your key info since this allows access to your AWS account.

Local Setup

My setup

- Windows 11 Pro (22H2)
- Git Bash <https://git-scm.com/download/win>
- Terraform v1.3.7 <https://releases.hashicorp.com/terraform/1.3.7/terraform_1.3.7_windows_amd64.zip>
- VS Code <https://code.visualstudio.com/>

After you download terraform, create a folder on `C:\terraform` to place the unzipped executable.

Update the path on Git Bash by adding this line to your ~/.bashrc file
`export PATH=$PATH:"/c/terraform/"`

Restart your terminal and verify the command ``terraform -version`` produces the following output:

```bash
$ terraform -version
Terraform v1.3.7
on windows_amd64
```

## Terraform Files

Create a new directory in your users folder (Example: C:\Users\Username\terraform). Open Git Bash and cd into that folder. Create the following files: main.tf and variables.tf.

Contents of the main.tf file

```terraform
# Configure the AWS Provider
provider "aws" {
  region = "us-east-1"
}
# Retrieve the list of AZs in the current AWS region
data "aws_availability_zones" "available" {}
data "aws_region" "current" {}
# Define the VPC
resource "aws_vpc" "vpc" {
  cidr_block = var.vpc_cidr
  tags = {
    Name        = var.vpc_name
    Environment = "demo_environment"
    Terraform   = "true"
  }
}
# Deploy the private subnets
resource "aws_subnet" "private_subnets" {
  for_each   = var.private_subnets
  vpc_id     = aws_vpc.vpc.id
  cidr_block = cidrsubnet(var.vpc_cidr, 8, each.value)
  availability_zone = tolist(data.aws_availability_zones.available.names)[
  each.value]
  tags = {
    Name      = each.key
    Terraform = "true"
  }
}
# Deploy the public subnets
resource "aws_subnet" "public_subnets" {
  for_each   = var.public_subnets
  vpc_id     = aws_vpc.vpc.id
  cidr_block = cidrsubnet(var.vpc_cidr, 8, each.value + 100)
  availability_zone = tolist(data.aws_availability_zones.available.
  names)[each.value]
  map_public_ip_on_launch = true
  tags = {
    Name      = each.key
    Terraform = "true"
  }
}
# Create route tables for public and private subnets
resource "aws_route_table" "public_route_table" {
  vpc_id = aws_vpc.vpc.id
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.internet_gateway.id
    # nat_gateway_id = aws_nat_gateway.nat_gateway.id
  }
  tags = {
    Name      = "demo_public_rtb"
    Terraform = "true"
  }
}
resource "aws_route_table" "private_route_table" {
  vpc_id = aws_vpc.vpc.id
  route {
    cidr_block = "0.0.0.0/0"
    # gateway_id = aws_internet_gateway.internet_gateway.id
    nat_gateway_id = aws_nat_gateway.nat_gateway.id
  }
  tags = {
    Name      = "demo_private_rtb"
    Terraform = "true"
  }
}
# Create route table associations
resource "aws_route_table_association" "public" {
  depends_on     = [aws_subnet.public_subnets]
  route_table_id = aws_route_table.public_route_table.id
  for_each       = aws_subnet.public_subnets
  subnet_id      = each.value.id
}
resource "aws_route_table_association" "private" {
  depends_on     = [aws_subnet.private_subnets]
  route_table_id = aws_route_table.private_route_table.id
  for_each       = aws_subnet.private_subnets
  subnet_id      = each.value.id
}
# Create Internet Gateway
resource "aws_internet_gateway" "internet_gateway" {
  vpc_id = aws_vpc.vpc.id
  tags = {
    Name = "demo_igw"
  }
}
# Create EIP for NAT Gateway
resource "aws_eip" "nat_gateway_eip" {
  vpc        = true
  depends_on = [aws_internet_gateway.internet_gateway]
  tags = {
    Name = "demo_igw_eip"
  }
}
# Create NAT Gateway
resource "aws_nat_gateway" "nat_gateway" {
  depends_on    = [aws_subnet.public_subnets]
  allocation_id = aws_eip.nat_gateway_eip.id
  subnet_id     = aws_subnet.public_subnets["public_subnet_1"].id
  tags = {
    Name = "demo_nat_gateway"
  }
}
```

Contents of the variables.tf file

```terraform
variable "aws_region" {
  type    = string
  default = "us-east-1"
}
variable "vpc_name" {
  type    = string
  default = "demo_vpc"
}
variable "vpc_cidr" {
  type    = string
  default = "10.0.0.0/16"
}
variable "private_subnets" {
  default = {
    "private_subnet_1" = 1
    "private_subnet_2" = 2
    "private_subnet_3" = 3
  }
}
variable "public_subnets" {
  default = {
    "public_subnet_1" = 1
    "public_subnet_2" = 2
    "public_subnet_3" = 3
  }
}
```

## Terraform Deployment

Here you will paste the values you saved earlier into the Git Bash console.

- export AWS_ACCESS_KEY_ID=``"<access key id>"``
- export AWS_SECRET_ACCESS_KEY=``"<secret access key>"``

Best practice is to always format your terraform files.

- ``terraform fmt``

Next initialize the backend

- ``terraform init``

Next run terraform plan to verify 18 resources will be created:

- ``terraform plan``

To create the infrastructure run terraform apply. You can either type 'yes' when prompted or automatically accept the results by using the -auto-approve flag.

- ``terraform apply``

Once you verified the files have been created by visiting the VPC service in the AWS console, you can delete the resource to avoid billing.

- ``terraform destroy -auto-approve``

Congratulations!
