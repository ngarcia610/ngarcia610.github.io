---
title: "Containers on AWS"
description: "ECS, Fargate, ECR and EKS"
date: 2023-08-28T18:04:05-04:00
lang: en
categories: ["Notes"]
tags: ["AWS"]
draft: True
---

What is Docker?

- A software development platform to deploy apps
- Apps are packaged in containers that can be run on any OS
- Apps run the same, regardless of where they're run:
  - Any machine
  - No compatibility issues
  - Predictible behavior
  - Less work
  - Easier to maintain and deploy
  - Works with any language, any OS, any technology
- Use cases: microservices architecture, lift-and-shift apps from on-premises to AWS cloud

Docker Images

- Stored in docker repositories
- [Docker Hub](https://hub.docker.com) Public repository
- Amazon ECR (Elastic Container Registry)
  - Private repo
  - Public repo [Amazon ECR Public Gallery](https://gallery.ecr.aws)

Docker Workflow

- Create a dockerfile
- Build a docker image
- Push it to a docker repo (docker hub or ECR)
- Pull the image
- Run the container

Amazon ECS - EC2 Launch Type

- ECS = Elastic Container Service
- Launch docker containers on AWS = launch ecs tasks on ecs clusters
- EC2 launch type: you must provision and maintain the infrastructure (ec2 instances)
- Each ec2 instance must run the ecs agent to register in the ecs cluster
- AWS takes care of starting / stopping containers

Amazon ECS - Fargate Launch Type

- Launch docker containers on aws
- No ec2 instances to manage (serverless)
- You create task definitions
- AWS runs the ecs tasks for you based on the CPU/RAM needed
- To scale, simply increase the number of tasks
