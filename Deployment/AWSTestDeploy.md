# Test Deployment Step-by-Step (Private Sponsor Viewing) 
## Overview

This guide establishes a complete AWS test environment for the OHS Remote application, including:

- Backend container registry (Amazon ECR)
- Secure networking layer (VPC, subnets, route tables, and security groups)
- Amazon RDS database
- Automated CI/CD pipeline
- Amazon ECS Fargate deployment

Once completed, every push to the `main` branch will automatically deploy an updated version of the application.

### Deployment Phases

1. Build and push the backend Docker image.
2. Configure AWS infrastructure.
3. Create the CI/CD pipeline and deploy the application.

---

## Phase 1 – Build and Push the Backend Docker Image to Amazon ECR

Amazon ECR (Elastic Container Registry) provides a private, managed container image repository for storing Docker images.

### Step 1 – Verify Docker Installation

```bash
docker --version
```

### Step 2 – Navigate to the Backend Project

Change into the root directory of the `ohs_backend` repository (the directory containing the `Dockerfile`).

### Step 3 – Create an Amazon ECR Repository

1. Open the AWS Console.
2. Navigate to **Elastic Container Registry (ECR)**.
3. Select **Create Repository**.
4. Configure:

| Setting | Value |
|----------|--------|
| Visibility | Private |
| Repository Name | `ohs-backend-repo` |
| Tag Mutability | Mutable |
| Scan on Push | Enabled |

5. Create the repository.
6. Copy the generated repository URI.

### Step 4 – Authenticate Docker to ECR

```bash
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <account-id>.dkr.ecr.us-east-1.amazonaws.com
```

### Step 5 – Build the Docker Image

```bash
docker build -t ohs-backend .
```

### Step 6 – Tag and Push the Image

```bash
docker tag ohs-backend:latest <repository-uri>:latest
docker push <repository-uri>:latest
```

---

## Phase 2 – AWS Infrastructure Setup

### Step 1 – Install and Configure AWS CLI

```bash
aws configure
```

Provide:

| Setting | Example |
|----------|----------|
| Access Key ID | IAM Access Key |
| Secret Access Key | IAM Secret Key |
| Region | `us-east-1` |
| Output Format | Leave Blank |

### Step 2 – Create IAM Roles

#### ECS Task Execution Role

Attach:

- AmazonECSTaskExecutionRolePolicy
- AmazonS3ReadOnlyAccess

Role Name:

```text
ecsTaskExecutionRole
```

#### CodePipeline Service Role

Attach:

- AWSCodePipeline_FullAccess
- AmazonECS_FullAccess

Role Name:

```text
CodePipelineServiceRole
```

### Step 3 – Create the VPC

| Setting | Value |
|----------|--------|
| Name | `ohs-remote-vpc` |
| CIDR | `10.0.0.0/16` |

### Step 4 – Create Subnets

| Subnet | Availability Zone | CIDR |
|----------|----------|----------|
| ohs-public-subnet-1 | us-east-1a | 10.0.1.0/24 |
| ohs-public-subnet-2 | us-east-1b | 10.0.2.0/24 |
| ohs-private-subnet-1 | us-east-1a | 10.0.3.0/24 |
| ohs-private-subnet-2 | us-east-1b | 10.0.4.0/24 |

### Step 5 – Create and Attach an Internet Gateway

| Setting | Value |
|----------|--------|
| Name | `ohs-igw` |
| Attach To | `ohs-remote-vpc` |

### Step 6 – Configure Route Tables

#### Public Route Table

| Setting | Value |
|----------|--------|
| Name | `ohs-public-rt` |
| Route | `0.0.0.0/0 → Internet Gateway` |

Associate both public subnets.

#### Private Route Table

| Setting | Value |
|----------|--------|
| Name | `ohs-private-rt` |
| Internet Route | None |

Associate both private subnets.

### Step 7 – Create Security Groups

#### Database Security Group

| Setting | Value |
|----------|--------|
| Name | `ohs-db-sg` |
| Inbound | MySQL (3306) from backend security group |

#### Backend Security Group

| Setting | Value |
|----------|--------|
| Name | `ohs-backend-sg` |
| Inbound | HTTP (80) |

### Step 8 – Create Amazon RDS MySQL

| Setting | Value |
|----------|--------|
| Instance Identifier | `ohs-db` |
| Engine | MySQL 8.x |
| Template | Free Tier |
| Database Name | `ohsdb` |
| Public Access | No |
| Security Group | `ohs-db-sg` |

---

## Phase 3 – ECS Fargate and CI/CD Deployment

### Step 9 – Create ECS Cluster

| Setting | Value |
|----------|--------|
| Cluster Name | `ohs-cluster` |
| Launch Type | AWS Fargate |

### Step 10 – Create Application Load Balancer

| Setting | Value |
|----------|--------|
| Name | `ohs-alb` |
| Type | Internet-Facing |
| Security Group | `ohs-alb-sg` |

#### Target Group

| Setting | Value |
|----------|--------|
| Name | `ohs-tg` |
| Type | IP |
| Health Check Path | `/health` |

### Step 11 – Create CodePipeline

| Stage | Service |
|---------|---------|
| Source | GitHub |
| Build | CodeBuild |
| Deploy | ECS |

Pipeline Name:

```text
ohs-pipeline
```

Build Project:

```text
ohs-build
```

### Step 12 – Create ECS Task Definition

Example configuration:

```json
{
  "family": "ohs-backend-task",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"]
}
```

### Step 13 – Create ECS Service

| Setting | Value |
|----------|--------|
| Service Name | `ohs-service` |
| Desired Tasks | 1 |
| Load Balancer | `ohs-alb` |
| Target Group | `ohs-tg` |

### Step 14 – Restrict Sponsor Access

Update the Application Load Balancer security group and replace:

```text
0.0.0.0/0
```

with:

```text
<SPONSOR_IP>/32
```

---

# Deployment Complete

The environment now includes:

- Amazon ECR
- Amazon ECS Fargate
- Amazon RDS MySQL
- Application Load Balancer
- AWS CodePipeline
- AWS CodeBuild
- VPC networking
- Security groups

---

# AWS Resource Summary

| AWS Service | Resource Name | Purpose |
|-------------|---------------|---------|
| Amazon ECR | ohs-backend-repo | Stores Docker images |
| Amazon VPC | ohs-remote-vpc | Isolated network |
| Public Subnets | ohs-public-subnet-1 / 2 | Load balancer hosting |
| Private Subnets | ohs-private-subnet-1 / 2 | Backend and database hosting |
| Internet Gateway | ohs-igw | Internet connectivity |
| Route Tables | ohs-public-rt / ohs-private-rt | Traffic routing |
| Security Groups | ohs-db-sg, ohs-backend-sg, ohs-alb-sg | Access control |
| Amazon RDS | ohs-db | MySQL database |
| ECS Cluster | ohs-cluster | Container orchestration |
| ECS Service | ohs-service | Backend runtime |
| Load Balancer | ohs-alb | Traffic distribution |
| Target Group | ohs-tg | Backend routing |
| CodePipeline | ohs-pipeline | Automated deployments |
| CodeBuild | ohs-build | Build automation |
| IAM Roles | ecsTaskExecutionRole, AWSCodePipelineServiceRole-ohs | Permissions management |