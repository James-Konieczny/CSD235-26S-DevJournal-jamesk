# 1.3 Production Deployment Guide (Public Launch)

## Overview

This section describes the upgrades required to transform the OHS Remote application from a private test deployment into a production-ready public web application.

### Production Upgrade Objectives

* Register and configure a public domain name
* Enable HTTPS using AWS Certificate Manager (ACM)
* Deploy the frontend using Amazon S3 and CloudFront
* Improve backend availability and scalability
* Expand the CI/CD pipeline
* Implement security hardening with AWS WAF
* Configure monitoring, alarms, and dashboards
* Optimize ongoing AWS costs

---

# Phase 1 – Domain Registration and DNS Management

Amazon Route 53 will be used for domain registration and DNS management.

## Step 1.1 – Register a Domain

1. Open the AWS Console.
2. Navigate to **Route 53**.
3. Select **Registered Domains** → **Register Domain**.
4. Search for an available domain name.
5. Complete the registration process.
6. Review pricing and purchase the domain.

### Result

Route 53 automatically creates:

* A Hosted Zone
* DNS Name Servers
* Domain registration records

---

## Step 1.2 – Create DNS Records

### Frontend DNS Record

Create an Alias A Record:

| Setting     | Value                   |
| ----------- | ----------------------- |
| Record Name | Root Domain or `www`    |
| Record Type | A Record                |
| Alias       | Enabled                 |
| Target      | CloudFront Distribution |

### Backend DNS Record

Create another Alias A Record:

| Setting     | Value                     |
| ----------- | ------------------------- |
| Record Name | `api`                     |
| Record Type | A Record                  |
| Alias       | Enabled                   |
| Target      | Application Load Balancer |

### Example

| Service     | URL                       |
| ----------- | ------------------------- |
| Frontend    | `https://example.com`     |
| Backend API | `https://api.example.com` |

---

# Phase 2 – SSL/TLS Certificates

AWS Certificate Manager (ACM) provides free, automatically renewing SSL certificates.

## Step 2.1 – Request CloudFront Certificate

> **Important:** CloudFront certificates must be created in **us-east-1**.

### Certificate Configuration

| Setting           | Value    |
| ----------------- | -------- |
| Certificate Type  | Public   |
| Validation Method | DNS      |
| Key Algorithm     | RSA 2048 |

### Domains

```text
example.com
api.example.com
```

Use **Create Records in Route 53** to automatically configure DNS validation.

Wait until the certificate status changes to:

```text
Issued
```

---

## Step 2.2 – Request ALB Certificate

Switch to the region hosting your infrastructure.

Request a second certificate for:

```text
api.example.com
```

Configuration:

| Setting           | Value    |
| ----------------- | -------- |
| Certificate Type  | Public   |
| Validation Method | DNS      |
| Key Algorithm     | RSA 2048 |

---

# Phase 3 – Frontend Production Hosting

The frontend will be deployed using:

* Amazon S3
* Amazon CloudFront
* AWS Certificate Manager

---

## Step 3.1 – Create Production S3 Bucket

| Setting       | Value                          |
| ------------- | ------------------------------ |
| Bucket Name   | `ohs-frontend-prod`            |
| Region        | Primary AWS Region             |
| Public Access | Managed through CloudFront OAC |

### Bucket Policy Example

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "cloudfront.amazonaws.com"
      },
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::YOUR_BUCKET_NAME/*"
    }
  ]
}
```

---

## Step 3.2 – Create CloudFront Distribution

### Origin Configuration

| Setting       | Value                       |
| ------------- | --------------------------- |
| Origin        | S3 Bucket                   |
| Origin Access | Origin Access Control (OAC) |

### Cache Behavior

| Setting                | Value                  |
| ---------------------- | ---------------------- |
| Viewer Protocol Policy | Redirect HTTP to HTTPS |
| Allowed Methods        | GET, HEAD              |

### Distribution Settings

| Setting                | Value                                                  |
| ---------------------- | ------------------------------------------------------ |
| Alternate Domain Names | example.com, [www.example.com](http://www.example.com) |
| SSL Certificate        | ACM Certificate                                        |
| Default Root Object    | index.html                                             |

---

## Step 3.3 – Extend the CI/CD Pipeline

### Create Frontend Build Project

Project Name:

```text
ohs-frontend-build
```

### Build Specification

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
      - npm run build

artifacts:
  files:
    - '**/*'
  base-directory: dist
```

### Pipeline Stages

| Stage              | Service    |
| ------------------ | ---------- |
| Source             | GitHub     |
| Build              | CodeBuild  |
| Deploy             | Amazon S3  |
| Cache Invalidation | CloudFront |

---

## Step 3.4 – Configure Frontend Environment Variables

Set the production API URL:

```env
VITE_API_BASE_URL=https://api.yourdomain.com
```

Configure this value within CodeBuild environment variables.

---

# Phase 4 – Backend Production Deployment

## Step 4.1 – Multi-AZ ECS Deployment

Update the ECS Service:

| Setting                 | Value     |
| ----------------------- | --------- |
| Minimum Healthy Percent | 100       |
| Maximum Percent         | 200       |
| Availability Zones      | Minimum 2 |

Ensure both private subnets are used.

---

## Step 4.2 – Configure ECS Auto Scaling

### Auto Scaling Configuration

| Setting       | Value           |
| ------------- | --------------- |
| Minimum Tasks | 2               |
| Maximum Tasks | 10              |
| Metric        | CPU Utilization |
| Target Value  | 70%             |

---

## Step 4.3 – Upgrade RDS to Multi-AZ

Modify the existing RDS instance.

### Configuration

| Setting           | Value    |
| ----------------- | -------- |
| Deployment Type   | Multi-AZ |
| Apply Immediately | Yes      |

### Benefits

* Automatic failover
* Increased availability
* No application code changes required

---

# Phase 5 – CI/CD Pipeline Expansion

## Step 5.1 – Add Manual Approval

### Recommended Pipeline Flow

```text
Source
   ↓
Build
   ↓
Deploy to Staging
   ↓
Manual Approval
   ↓
Deploy to Production
```

### Manual Approval Stage

| Setting     | Value              |
| ----------- | ------------------ |
| Stage Name  | ProductionApproval |
| Action Type | Manual Approval    |

---

## Step 5.2 – Separate Environments

Recommended deployment strategy:

| Environment | ECS Service         |
| ----------- | ------------------- |
| Staging     | ohs-service-staging |
| Production  | ohs-service         |

---

# Phase 6 – Security Hardening (AWS WAF)

AWS WAF protects against common web attacks and malicious traffic.

## Step 6.1 – Create a Web ACL

### Configuration

| Setting       | Value              |
| ------------- | ------------------ |
| Name          | ohs-production-waf |
| Resource Type | CloudFront         |

### Managed Rule Groups

| Rule Group                            | Purpose                           |
| ------------------------------------- | --------------------------------- |
| AWSManagedRulesCommonRuleSet          | OWASP Top 10 Protection           |
| AWSManagedRulesAmazonIpReputationList | Blocks Known Malicious IPs        |
| AWSManagedRulesKnownBadInputsRuleSet  | Blocks Malicious Request Patterns |

---

## Step 6.2 – Associate WAF with CloudFront

Initially configure rules in:

```text
Count Mode
```

After validation, switch to:

```text
Block Mode
```

---

# Phase 7 – Monitoring and Alarms

Amazon CloudWatch provides operational visibility into application performance.

---

## Step 7.1 – Enable Detailed Monitoring

### RDS

Enable:

```text
Enhanced Monitoring (1 Second)
```

### ECS

Ensure task definitions use:

```text
awslogs
```

for CloudWatch logging.

---

## Step 7.2 – Create CloudWatch Dashboard

Dashboard Name:

```text
OHS-Production-Dashboard
```

### Recommended Metrics

| Service | Metrics                             |
| ------- | ----------------------------------- |
| ECS     | CPU Utilization, Memory Utilization |
| RDS     | Connections, CPU Usage, Free Memory |
| ALB     | Request Count, Response Time        |

---

## Step 7.3 – Configure Alarms

### SNS Topic

```text
ohs-alarms
```

### ECS CPU Alarm

| Setting           | Value           |
| ----------------- | --------------- |
| Metric            | CPU Utilization |
| Threshold         | > 80%           |
| Evaluation Period | 2 Minutes       |

### Billing Alarm

| Setting   | Value             |
| --------- | ----------------- |
| Metric    | Estimated Charges |
| Threshold | > $200 USD        |

---

# Phase 8 – Cost Optimization

## Recommended Practices

### Elastic IP Addresses

Release unused Elastic IPs.

### Database Optimization

Consider:

```text
Amazon Aurora Serverless v2
```

for variable workloads.

### S3 Lifecycle Policies

Automatically transition old artifacts to:

* S3 Glacier Instant Retrieval
* S3 Glacier Flexible Retrieval

### AWS Budgets

Configure budget alerts in addition to CloudWatch alarms.

---

# Production Deployment Summary

| Resource Category | Test Deployment | Production Deployment                  |
| ----------------- | --------------- | -------------------------------------- |
| Domain & DNS      | Not Configured  | Route 53 Hosted Zone                   |
| SSL/TLS           | None            | ACM Certificates                       |
| Frontend Hosting  | S3 Bucket       | S3 + CloudFront                        |
| Backend Compute   | Single ECS Task | Multi-AZ ECS Service                   |
| Database          | Single-AZ RDS   | Multi-AZ RDS                           |
| CI/CD             | Backend Only    | Frontend + Backend + Approval Workflow |
| Security          | IP Whitelisting | AWS WAF                                |
| Monitoring        | Basic Metrics   | Dashboards, Alarms, SNS Notifications  |

## Final Production Architecture

**Presentation Tier**

* Route 53
* CloudFront
* Amazon S3
* AWS Certificate Manager

**Application Tier**

* Application Load Balancer
* Amazon ECS
* AWS Fargate
* Amazon ECR

**Data Tier**

* Amazon RDS MySQL (Multi-AZ)

**Security & Monitoring**

* AWS WAF
* Amazon CloudWatch
* Amazon SNS

**CI/CD**

* GitHub
* AWS CodePipeline
* AWS CodeBuild
* Amazon S3 Deployment
* ECS Rolling Deployments
