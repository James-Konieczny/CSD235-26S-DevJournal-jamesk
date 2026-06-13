# 1.4 CI/CD Setup (CodePipeline, ECR, CodeBuild)

## Overview

This section describes how to build a complete Continuous Integration and Continuous Deployment (CI/CD) pipeline for the OHS Remote platform.

By the end of this setup, every push to the `main` branch of either repository will automatically:

* Build and test application code
* Create and publish backend Docker images
* Deploy backend updates to Amazon ECS Fargate
* Deploy frontend updates to Amazon S3
* Invalidate the CloudFront cache
* Optionally require manual approval before production deployment

### Technologies Used

| Service            | Purpose                   |
| ------------------ | ------------------------- |
| Amazon ECR         | Container image registry  |
| AWS CodeBuild      | Build and test automation |
| AWS CodePipeline   | CI/CD orchestration       |
| Amazon ECS Fargate | Backend deployment target |
| Amazon S3          | Frontend hosting          |
| Amazon CloudFront  | CDN and content delivery  |
| GitHub             | Source code repository    |

---

# Prerequisites

Before beginning, ensure the following resources already exist:

* AWS account with administrative access
* AWS CLI configured (`aws configure`)
* Git installed locally
* Docker installed locally
* Backend GitHub repository (`ohs_backend`)
* Frontend GitHub repository (`ohs_frontend`)
* ECS cluster and service
* Application Load Balancer
* Target Group
* Production S3 bucket
* CloudFront distribution

> **Note:** If these resources do not exist, complete the Test Deployment and Production Deployment setup guides first.

---

# Part A – Amazon ECR (Container Registry)

Amazon Elastic Container Registry (ECR) provides a private Docker image repository used by ECS deployments.

## Step A1 – Create an ECR Repository

Navigate to:

```text
AWS Console → Elastic Container Registry (ECR)
```

Select **Create Repository** and configure the following:

| Setting         | Value         |
| --------------- | ------------- |
| Visibility      | Private       |
| Repository Name | `ohs-backend` |
| Tag Mutability  | Mutable       |
| Scan on Push    | Enabled       |

After creation, note the Repository URI.

### Example Repository URI

```text
123456789012.dkr.ecr.us-east-1.amazonaws.com/ohs-backend
```

---

## Step A2 – Test Docker Authentication (Optional)

Authenticate Docker against ECR:

```bash
aws ecr get-login-password --region us-east-1 \
| docker login \
--username AWS \
--password-stdin 123456789012.dkr.ecr.us-east-1.amazonaws.com
```

Build the image:

```bash
cd /path/to/ohs_backend
docker build -t ohs-backend .
```

Tag the image:

```bash
docker tag ohs-backend:latest \
123456789012.dkr.ecr.us-east-1.amazonaws.com/ohs-backend:latest
```

Push the image:

```bash
docker push \
123456789012.dkr.ecr.us-east-1.amazonaws.com/ohs-backend:latest
```

---

# Part B – AWS CodeBuild

CodeBuild automates application builds and deployment packaging.

## Step B1 – Create the CodeBuild Service Role

Navigate to:

```text
IAM → Roles → Create Role
```

### Role Configuration

| Setting        | Value       |
| -------------- | ----------- |
| Trusted Entity | AWS Service |
| Use Case       | CodeBuild   |

### Required Policies

* AmazonEC2ContainerRegistryPowerUser
* AmazonS3FullAccess
* CloudWatchLogsFullAccess
* AWSCloudFormationReadOnlyAccess (Optional)
* CodeBuildDeveloperAccess

### Role Name

```text
CodeBuildServiceRole-ohs
```

---

## Step B2 – Backend Build Project

### Project Information

| Setting         | Value                     |
| --------------- | ------------------------- |
| Project Name    | `ohs-backend-build`       |
| Source Provider | GitHub                    |
| Repository      | `Capstone347/ohs_backend` |
| Branch          | `main`                    |

### Environment

| Setting          | Value                      |
| ---------------- | -------------------------- |
| Operating System | Amazon Linux 2             |
| Runtime          | Standard                   |
| Environment Type | Linux                      |
| Privileged Mode  | Enabled                    |
| Service Role     | `CodeBuildServiceRole-ohs` |

### Build Specification

Create a file named:

```text
buildspec-backend.yml
```

```yaml
version: 0.2

phases:
  pre_build:
    commands:
      - echo Logging in to Amazon ECR...
      - aws ecr get-login-password --region $AWS_DEFAULT_REGION | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com

  build:
    commands:
      - echo Building Docker image...
      - docker build -t $IMAGE_REPO_NAME:latest .
      - docker tag $IMAGE_REPO_NAME:latest $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$IMAGE_REPO_NAME:latest

  post_build:
    commands:
      - echo Pushing Docker image...
      - docker push $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$IMAGE_REPO_NAME:latest
      - printf '[{"name":"ohs-backend-container","imageUri":"%s"}]' $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$IMAGE_REPO_NAME:latest > imagedefinitions.json

artifacts:
  files:
    - imagedefinitions.json
```

### Required Environment Variables

| Variable           | Example      |
| ------------------ | ------------ |
| AWS_ACCOUNT_ID     | 123456789012 |
| IMAGE_REPO_NAME    | ohs-backend  |
| AWS_DEFAULT_REGION | us-east-1    |

---

## Step B3 – Frontend Build Project

### Project Information

| Setting      | Value                      |
| ------------ | -------------------------- |
| Project Name | `ohs-frontend-build`       |
| Repository   | `Capstone347/ohs_frontend` |
| Service Role | `CodeBuildServiceRole-ohs` |

### Build Specification

Create:

```text
buildspec-frontend.yml
```

```yaml
version: 0.2

phases:
  install:
    runtime-versions:
      nodejs: 18
    commands:
      - npm install

  build:
    commands:
      - VITE_API_BASE_URL=https://api.yourdomain.com npm run build

artifacts:
  files:
    - '**/*'
  base-directory: dist
  discard-paths: no
```

> Replace `https://api.yourdomain.com` with your production API endpoint.

---

## Step B4 – CloudFront Cache Invalidation (Optional)

Create a third CodeBuild project:

```text
ohs-cloudfront-invalidate
```

### Build Specification

```yaml
version: 0.2

phases:
  build:
    commands:
      - aws cloudfront create-invalidation \
        --distribution-id YOUR_DISTRIBUTION_ID \
        --paths "/*"
```

---

# Part C – AWS CodePipeline

CodePipeline orchestrates source retrieval, builds, and deployments.

---

## Step C1 – Create the CodePipeline Service Role

### Required Policies

* AWSCodePipeline_FullAccess
* AmazonECS_FullAccess
* AWSCodeBuildDeveloperAccess
* AmazonS3FullAccess
* CloudWatchLogsFullAccess

### Role Name

```text
CodePipelineServiceRole-ohs
```

---

## Step C2 – Create the Artifact Bucket

Create a dedicated S3 bucket:

| Setting          | Value                          |
| ---------------- | ------------------------------ |
| Bucket Name      | `ohs-pipeline-artifacts-xxxxx` |
| Public Access    | Blocked                        |
| Object Ownership | ACLs Disabled                  |

---

## Step C3 – Create the Pipeline

### Pipeline Information

| Setting        | Value                         |
| -------------- | ----------------------------- |
| Pipeline Name  | `ohs-pipeline`                |
| Service Role   | `CodePipelineServiceRole-ohs` |
| Artifact Store | S3 Artifact Bucket            |

---

### Source Stage

#### Backend Source

| Setting         | Value                   |
| --------------- | ----------------------- |
| Action Name     | BackendSource           |
| Repository      | Capstone347/ohs_backend |
| Branch          | main                    |
| Output Artifact | BackendSourceArtifact   |

#### Frontend Source

| Setting         | Value                    |
| --------------- | ------------------------ |
| Action Name     | FrontendSource           |
| Repository      | Capstone347/ohs_frontend |
| Branch          | main                     |
| Output Artifact | FrontendSourceArtifact   |

---

### Build Stage

#### Backend Build

| Setting         | Value                 |
| --------------- | --------------------- |
| Action Name     | BackendBuild          |
| Project         | ohs-backend-build     |
| Input Artifact  | BackendSourceArtifact |
| Output Artifact | BackendBuildArtifact  |

#### Frontend Build

| Setting         | Value                  |
| --------------- | ---------------------- |
| Action Name     | FrontendBuild          |
| Project         | ohs-frontend-build     |
| Input Artifact  | FrontendSourceArtifact |
| Output Artifact | FrontendBuildArtifact  |

---

### Frontend Deployment Stage

#### Deploy to S3

| Setting        | Value                      |
| -------------- | -------------------------- |
| Action Name    | DeployToS3                 |
| Bucket         | Production Frontend Bucket |
| Input Artifact | FrontendBuildArtifact      |

Enable:

```text
Extract file before deploy
```

#### CloudFront Invalidation

| Setting     | Value                     |
| ----------- | ------------------------- |
| Action Name | InvalidateCloudFront      |
| Project     | ohs-cloudfront-invalidate |

---

### Backend Deployment Stage

#### ECS Deployment

| Setting            | Value       |
| ------------------ | ----------- |
| Action Name        | DeployToECS |
| Cluster            | ohs-cluster |
| Service            | ohs-service |
| Deployment Timeout | 10 Minutes  |

---

## Step C4 – Add Manual Approval (Recommended)

Insert an approval stage between Build and Deploy.

### Approval Configuration

| Setting     | Value           |
| ----------- | --------------- |
| Stage Name  | Approval        |
| Action Name | WaitForApproval |
| Action Type | Manual Approval |

### Recommended Pipeline Flow

```text
Source
   ↓
Build
   ↓
Approval
   ↓
Deploy Frontend
   ↓
Deploy Backend
```

---

# Part D – Running the Pipeline

Navigate to:

```text
CodePipeline → ohs-pipeline
```

Select:

```text
Release Change
```

Monitor each stage:

1. Source
2. Build
3. Approval (Optional)
4. Frontend Deployment
5. Backend Deployment

If failures occur, review CloudWatch logs from the affected CodeBuild project.

---

# Part E – Ongoing Operation

Once configured, the pipeline operates automatically.

### Backend Changes

1. Build Docker image
2. Push image to ECR
3. Deploy to ECS
4. Perform rolling update

### Frontend Changes

1. Build React application
2. Upload to S3
3. Invalidate CloudFront cache
4. Publish globally

---

# CI/CD Component Summary

| Component       | Name                        | Purpose                |
| --------------- | --------------------------- | ---------------------- |
| ECR Repository  | ohs-backend                 | Stores Docker images   |
| CodeBuild       | ohs-backend-build           | Backend build process  |
| CodeBuild       | ohs-frontend-build          | Frontend build process |
| CodeBuild       | ohs-cloudfront-invalidate   | Cache invalidation     |
| CodePipeline    | ohs-pipeline                | CI/CD orchestration    |
| Artifact Bucket | ohs-pipeline-artifacts-xxx  | Build artifact storage |
| IAM Role        | CodeBuildServiceRole-ohs    | Build permissions      |
| IAM Role        | CodePipelineServiceRole-ohs | Pipeline permissions   |

---

# Troubleshooting

| Issue                        | Likely Cause                      | Resolution                                   |
| ---------------------------- | --------------------------------- | -------------------------------------------- |
| AccessDenied in CodeBuild    | Missing ECR permissions           | Attach `AmazonEC2ContainerRegistryPowerUser` |
| `docker: command not found`  | Privileged mode disabled          | Enable privileged mode in CodeBuild          |
| ECS deployment fails         | Incorrect cluster or service name | Verify ECS configuration                     |
| Frontend changes not visible | CloudFront cache not invalidated  | Run invalidation manually                    |
| GitHub connection fails      | Expired OAuth token               | Reconnect GitHub source                      |

---

## Outcome

Upon completion, OHS Remote will have a fully automated CI/CD pipeline that:

* Builds and tests code automatically
* Publishes backend container images to Amazon ECR
* Deploys backend updates to Amazon ECS Fargate
* Deploys frontend updates to Amazon S3
* Refreshes CloudFront caches automatically
* Supports manual approvals for production releases
* Enables reliable, repeatable deployments with minimal operational effort
