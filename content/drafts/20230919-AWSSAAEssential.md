---
title: "AWS SAA Need to Know"
description: "Essential concepts to know for the exam"
date: 2023-09-19T18:04:05-04:00
lang: en
categories: ["Notes"]
tags: ["AWS"]
draft: True
---

## Domain 1: Design Secure Architectures

Design secure access to AWS resources

- Secure the account root user with MFA, and create a new restricted user to use instead of the root account.
- Principle of least privilege
- Creating IAM users, groups, and roles
- Best practices for handling credentials
- AWS security token service
- Federation (Use an IAM role), configure AD to use IAM users and roles
- Control access to application api
- Read and interpret policy documents (identity based versus bucket policies)
- Traceability (cloudtrail, cloudwatch, aws organizations, control tower)

Design secure workloads and applications

- Amazon VPC (default and custom) default settings
- Design secure vpc architecture (secure application tiers)
- Security groups, NACL, route tables, NAT gateways
- Public and private subnets
- PrivateLink, scale vpcs to 10s or hundreds of vpcs
- Security design strategies (GuardDuty, Macie, Cognito, Secrets Manager, Shield, Shield Adv)

Determine appropriate data security controls

- Securing access to encryption keys
- Encryption at rest versus encryption at transit
- AWS Certificate Manager for SSL certs and renewals
- AWS KMS for generating, storing, and securing access to keys
- S3 server side versus client side encryption
- Data backups and replication
- Often times the best solution is the most secure solution
- EBS, RDS, Aurora, Neptune snapshots, dynamodb backup
- EFS backup when using AWS backup
- DocumentDB
- S3 cross-region replication for asynchronous copy

Important AWS articles for this domain

- Managing Your AWS Account
- What is IAM?
- What is Amazon VPC?
- Security in Amazon Virtual Private Cloud
- Security, Identity, and Compliance
- Data Encryption
- Amazon EBS Snapshots
- Point-in-Time Recovery for DynamoDB
- Backing up and Restoring an Amazon RDS DB Instance
- Creating a DB Cluster Snapshot
- Using AWS Backup to Back up and Restore Amazon EFS File Systems
- Amazon Redshift Snapshots and Backups
- Overview of Backing up and Restoring a Neptune DB Cluster
- Backing up and Restoring in Amazon DocumentDB
- Amazon S3 Cross-Region Replication (CRR)
- AWS Backup
- Amazon Elastic Block Store (EBS)
- Amazon EC2 instances
- Amazon Relational Database Service (Amazon RDS) databases (including Amazon Aurora databases)
- Amazon DynamoDB
- Amazon Elastic File System (Amazon EFS)
- AWS Storage Gateway
- Amazon FSx for Windows File Server and Amazon FSx for Lustre

## Domain 2: Design Resilient Architectures

Design scalable and loosely coupled architectures

- Vertical versus horizontal scaling
- Know when to use which compute workload (EC2, EKS, ECS, Lambda)
- Database workloads and read replicas versus caching
- Understand serverless and Lambda
- Scaling message consumers and producers horizontally
- Use asynchronous workloads and decoupling
- API Gateway, Transfer Family, SQS, Secrets Manager, ALB, SQS, Fargate

Design highly available and/or fault-tolerant architectures

- 
