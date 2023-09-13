---
title: "Terraform Up and Running"
description: "3rd Edition by Yevginiy Brikman, Chapters 1-3"
date: 2023-09-11T18:04:05-04:00
lang: en
categories: ["Notes"]
tags: ["DevOps"]
draft: True
---

## Chapter 1 - Why Terraform

What is DevOps?

- Software delivery used to consist of developers writing code, and operations (sysadmins) handling hardware and server configuration.
- Now that companies are moving to Cloud providers instead of hosting their own datacenters, everyone works on software.
- The process of combining Dev and Ops teams to make the software delivery process more efficient is known as DevOps.
- Devops consists of four core values: CAMS
  - Culture
  - Automation
  - Measurement
  - Sharing

What is Infrastructure as Code?

- Code used to define, deploy, update, and destroy infrastructure.
- All aspects of operations are treated as software.
- Five categories of IaC:
  - Ad hoc scripts
    - A sequence of instructions executed on a server.
    - Examples: bash, ruby, python, powershell
  - Configuration management tools
    - Install and manage server on existing servers.
    - Examples: chef, puppet, ansible
    - Similar to ad hoc scripts but adds coding conventions, idempotence, and distribution
  - Server Templating tools
    - Capture an image of a server that includes OS, software, files, and configurations.
    - Examples: Docker, Packer, and Vagrant
    - Images can be a VM or a Container
  - Orchestration tools
    - Manage your vms and containers.
    - Roll out updates, monitor health, load balancing, service discovery, auto scaling, auto healing
    - Examples: Kubernetes, Marathon/Mesos, Amazon ECS, Docker Swarm, Nomad
    - Cloud providers have native Kubernetes support: Amazon EKS, Google GKE, Azure AKS
  - Provisioning tools
    - Creates the servers.
    - Examples: TERRAFORM, AWS Cloud Formation, Openstack Heat, Pulumi

What are the benefits of IaC?

- Self-service: Developers can initiate their own deployments as necessary.
- Speed and safety: avoids human error and deploys faster.
- Documentation: infrastructure is defined in source files anyone can read.
- Version Control: history to help debug issues
- Validation: you can review and automate testing
- Reuse: you don't have to deploy from scratch
- Happiness: focus on work with meaning instead of repetitive tasks

How does Terraform work?

- There is a single, compiled binary called "terraform" written in Go, that makes API calls to providers.
- Terraform configurations: text files that specify what infrastructure you want to create
- When someone then needs to change the infrastructure, they:
  - Make changes to the terraform configuration files
  - validate those changes through automated tests and code reviews
  - commit the updated code to version control
  - run `terraform apply` to apply the changes (make the API calls)

## Chapter 2 - Getting Started with Terraform

Setup your AWS account

- [https://aws.amazon.com](Sign up for an account. This becomes the root account.)
- Create a new user in iam
- Create a new group and attach the policy `AdministratorAccess`
- Click the user and go to the security credentials tab and select create access key
- Save the access key credentials, they are only shown once.
- Install the AWS CLI (using Fedora):
  - Verify `unzip` is installed
  - `curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"`
  - `unzip awscliv2.zip`
  - `sudo ./aws/install`
  - `aws configure`
  - Enter your access key id, secret access key, region name, and output format
  - `aws configure list-profiles` should have one profile named default
  - `~./aws/credentials` is the default location for the newly created config

Install Terraform

- `sudo dnf install -y dnf-plugins-core` verify config-manager is installed to manage repos
- `sudo dnf config-manager --add-repo https://rpm.releases.hashicorp.com/fedora/hashicorp.repo` add Hashicorp repo
- `sudo dnf -y install terraform` install terraform
- `terraform -v` verify installation

Deploy a Server

- Create a main.tf file:

```HCL
provider "aws" {
  region = "us-east-1"
}

resource "aws_instance" "example" {
  ami = "ami-0261755bbcb8c4a84"
  instance_type = "t2.micro"
  subnet_id = "subnet-<your subnet_id>"

  tags = {
    Name = "terraform-example"
  }
}
```

- Navigate to that directory and run `terraform init`
- Use `terraform plan` to view the expected output
- Use `terraform apply` to create the resources after confirming with `yes`
- Confirm the resources were created using the management console

Deploy a Web Server

- Below is the example code for main.tf:

```HCL
terraform {
  required_version = ">= 1.0.0, < 2.0.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 4.0"
    }
  }
}

provider "aws" {
  region = "us-east-1"
}

resource "aws_instance" "example" {
  ami                    = "ami-0261755bbcb8c4a84"
  instance_type          = "t2.micro"
  subnet_id              = "subnet-0f36dcdc980d594e8"
  vpc_security_group_ids = [aws_security_group.instance.id]

  user_data = <<-EOF
              #!/bin/bash
              echo "Hello, World" > index.html
              nohup busybox httpd -f -p 8080 &
              EOF

  user_data_replace_on_change = true

  tags = {
    Name = "terraform-example"
  }
}

resource "aws_security_group" "instance" {

  name = var.security_group_name

  ingress {
    from_port   = 8080
    to_port     = 8080
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

variable "security_group_name" {
  description = "The name of the security group"
  type        = string
  default     = "terraform-example-instance"
}

output "public_ip" {
  value       = aws_instance.example.public_ip
  description = "The public IP of the Instance"
}
```

- `vpc_security_group_ids` creates a dependency where the security group resource is made first
- `user_data_replace_on_change` means destroy and recreate the resource when changed (needed to update user_data)
- `user_data` commands that run at launch of the ec2 instance (here we use busybox to create a simple web server)
- The new resource block adds a security group
- The AMI is unique to each region. The book uses us-east-2, so it has a different ami id.
- Once it is deployed use `curl http://<public ip>:8080` to return the value "Hello, World"

Deploy a Configurable Web Server

- DRY (don't repeat yourself), use variables for repeated values
- Example of a variable that checks if a number is present:

```HCL
variable "number_example" {
  description = "An example of a number variable in Terraform"
  type = number
  default = 85
}
```

- Variable blocks have these optional parameters:
  - description: friendly text
  - default: fallback value if none is given
  - type: enforce type constraints
  - validation: more nuanced than type checks
  - sensitive: if true, terraform does not log it on `plan` or `apply`

Deploy a Cluster of Web Servers

- Use an auto-scaling group (ASG) to create a cluster of instances
- Create a launch configuration
- `aws_launch_configuration` is similar to `aws_instance` with some differences:
  - `ami` becomes `image_id`
  - `vpc_security_group` becomes `security_groups`
- Tags are moved to `aws_autoscaling_group` resource
- Example:

```HCL
resource "aws_autoscaling_group" "example" {
  launch_configuration = aws_launch_configuration.example.name
  vpc_zone_identifier  = data.aws_subnets.default.ids

  target_group_arns = [aws_lb_target_group.asg.arn]
  health_check_type = "ELB"

  min_size = 2
  max_size = 10

  tag {
    key                 = "Name"
    value               = "terraform-asg-example"
    propagate_at_launch = true
  }
}
```

Deploy a Load Balancer

- AWS has 3 types of load balancers:
  - ALB application load balancer (http(s) traffic)
  - NLB network load balancer (tcp, udp, tls traffic; for extreme networking)
  - CLB classic load balancer (legacy)
- An ALB consists of: listeners, listener rules, and target groups
- During validation remember to have at least 2 subnets in different AZs
- Example ALB terraform:

```HCL
resource "aws_lb" "example" {

  name = var.alb_name

  load_balancer_type = "application"
  subnets            = data.aws_subnets.default.ids
  security_groups    = [aws_security_group.alb.id]
}

resource "aws_lb_listener" "http" {
  load_balancer_arn = aws_lb.example.arn
  port              = 80
  protocol          = "HTTP"

  # By default, return a simple 404 page
  default_action {
    type = "fixed-response"

    fixed_response {
      content_type = "text/plain"
      message_body = "404: page not found"
      status_code  = 404
    }
  }
}

resource "aws_lb_target_group" "asg" {

  name = var.alb_name

  port     = var.server_port
  protocol = "HTTP"
  vpc_id   = data.aws_vpc.default.id

  health_check {
    path                = "/"
    protocol            = "HTTP"
    matcher             = "200"
    interval            = 15
    timeout             = 3
    healthy_threshold   = 2
    unhealthy_threshold = 2
  }
}

resource "aws_lb_listener_rule" "asg" {
  listener_arn = aws_lb_listener.http.arn
  priority     = 100

  condition {
    path_pattern {
      values = ["*"]
    }
  }

  action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.asg.arn
  }
}

resource "aws_security_group" "alb" {

  name = var.alb_security_group_name

  # Allow inbound HTTP requests
  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  # Allow all outbound requests
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

Cleanup

- `terraform destroy` terminates and removes all resources

## Chapter 3 - How to Manage Terraform State
