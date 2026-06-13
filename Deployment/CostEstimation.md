# 1.5 Cost Estimation & Free Tier Usage

## Overview

This section provides a detailed cost analysis of the AWS infrastructure proposed throughout this deployment document.

Cost estimates are separated into:

* **Test Deployment** (Private Sponsor Access)
* **Production Deployment – Low Traffic**
* **Production Deployment – Medium Traffic**

All estimates are based on **AWS US East (N. Virginia)** pricing and are accurate as of **June 2026**.

> **Important:** AWS changed its Free Tier program on July 15, 2025. Your available credits and free resources depend on when your AWS account was created.

---

# AWS Free Tier Overview

## Accounts Created Before July 15, 2025

Legacy 12-Month Free Tier:

* 750 hours/month of eligible resources
* Free RDS Single-AZ instances
* Free CodeBuild minutes
* Various service-specific allowances

## Accounts Created On or After July 15, 2025

AWS Free Plan:

* $100 AWS credits
* Valid for 6 months
* Potential for additional promotional credits
* Limited service availability

> Verify which program applies to your account before deploying production workloads.

---

# Test Deployment Cost Estimate

The test deployment is designed for sponsor review and validation purposes with minimal traffic and resource consumption.

## Monthly Cost Breakdown

| Service                   | Configuration                          | Estimated Monthly Cost | Notes                        |
| ------------------------- | -------------------------------------- | ---------------------- | ---------------------------- |
| RDS MySQL                 | Single-AZ `db.t3.micro`, 20 GB storage | $0.00                  | Covered by legacy free tier  |
| ECS Fargate               | 1 task, 0.25 vCPU, 0.5 GB RAM          | ~$8.50                 | No free tier available       |
| Application Load Balancer | 1 ALB                                  | ~$16.40                | Base hourly charge           |
| S3 Frontend Hosting       | Static assets (~10 MB)                 | $0.00                  | Negligible storage cost      |
| CloudFront                | Not used                               | $0.00                  | Added only in production     |
| Amazon ECR                | 500 MB image storage                   | ~$0.05                 | Approximately $0.10/GB-month |
| CodePipeline              | 1 pipeline                             | $0.00–$1.00            | First 30 days free           |
| CodeBuild                 | ~10 build minutes/month                | $0.00                  | Within free allocation       |
| Secrets Manager           | Not used                               | $0.00                  | Production only              |
| Route 53                  | Not used                               | $0.00                  | No custom domain             |
| CloudWatch Logs           | Minimal logging                        | $0.00                  | Within free allowance        |
| Data Transfer             | Sponsor access only                    | $0.00                  | Negligible usage             |

## Estimated Monthly Total

| Scenario                 | Cost    |
| ------------------------ | ------- |
| First Month              | ~$0.00  |
| Ongoing Test Environment | ~$25.00 |

### Key Cost Drivers

* ECS Fargate (~$8.50/month)
* Application Load Balancer (~$16.40/month)

---

## Test Environment Cost Optimization

To reduce costs during testing:

* Stop ECS services outside testing hours
* Stop RDS instances when not needed
* Delete unused ECR images
* Disable unused pipelines

> This approach is only recommended for non-production environments.

---

# Production Deployment Cost Estimate (Low Traffic)

The production deployment includes:

* Multi-AZ database deployment
* High-availability ECS services
* CloudFront CDN
* Route 53
* AWS WAF
* Secrets Manager
* Enhanced monitoring

Assumptions:

* Few hundred visitors per day
* Less than 10 GB monthly data transfer
* Low API usage

---

## Monthly Cost Breakdown

| Service                   | Configuration                     | Estimated Monthly Cost |
| ------------------------- | --------------------------------- | ---------------------- |
| RDS MySQL                 | Multi-AZ `db.t3.micro`            | ~$23.36                |
| ECS Fargate               | 2 tasks (0.5 vCPU / 1 GB each)    | ~$35.74                |
| Application Load Balancer | 1 ALB + light LCU usage           | ~$18.40                |
| S3 Frontend Hosting       | Static assets                     | $0.00                  |
| CloudFront                | CDN                               | $0.00                  |
| Amazon ECR                | Image storage                     | ~$0.05                 |
| CodePipeline              | 1 active pipeline                 | ~$1.00                 |
| CodeBuild                 | ~30 build minutes/month           | $0.00                  |
| Secrets Manager           | 5 secrets                         | ~$2.00                 |
| Route 53                  | Hosted zone + DNS queries         | ~$0.90                 |
| AWS WAF                   | Managed rule groups               | ~$5.00                 |
| CloudWatch Logs           | Moderate logging                  | ~$2.50                 |
| ACM Certificates          | HTTPS certificates                | $0.00                  |
| Data Transfer             | Within CloudFront free allocation | $0.00                  |

## Estimated Monthly Total

### Low-Traffic Production

```text
~$87.00/month
```

---

# Production Deployment Cost Estimate (Medium Traffic)

Medium traffic assumes:

* 5,000–10,000 visitors per day
* 150,000–300,000 visitors per month
* Increased request volume and logging

---

## Monthly Cost Breakdown

| Service                   | Low Traffic | Medium Traffic | Notes                       |
| ------------------------- | ----------- | -------------- | --------------------------- |
| RDS MySQL                 | $23.36      | $23.36         | Fixed instance cost         |
| ECS Fargate               | $35.74      | $35.74         | No scaling assumed          |
| Application Load Balancer | $18.40      | $24.40         | Increased LCU usage         |
| CloudFront                | $0.00       | $8.50          | Additional bandwidth        |
| Route 53                  | $0.90       | $2.50          | Higher DNS query volume     |
| AWS WAF                   | $5.00       | $8.00          | Additional rule evaluations |
| CloudWatch Logs           | $2.50       | $10.00         | Increased log volume        |
| Other Services            | ~$3.00      | ~$3.00         | Minimal change              |

## Estimated Monthly Total

### Medium-Traffic Production

```text
~$135.00/month
```

---

# Free Tier Notes and Warnings

## Account Creation Date Matters

### Accounts Created Before July 15, 2025

* Legacy 12-month free tier
* Resource-based allowances
* Free RDS Single-AZ instances

### Accounts Created After July 15, 2025

* $100 AWS credits
* Six-month duration
* Account closure if credits expire and billing is not enabled

---

## Multi-AZ RDS Is Not Free

Once Multi-AZ is enabled:

* Free-tier eligibility no longer applies
* Both primary and standby instances are billed

---

## CodePipeline Pricing

* First 30 days free
* Approximately $1/month per active pipeline

---

## CodeBuild Free Minutes

Included:

* 100 build minutes/month
* Applies to both frontend and backend builds

Additional usage:

```text
$0.005 per minute
```

---

## CloudFront Free Tier

CloudFront includes:

```text
1 TB/month outbound data transfer
```

This benefit:

* Does not expire
* Applies to all AWS customers
* Is independent of account age

---

## Billing Alarms Are Essential

Always configure:

* AWS Budgets
* CloudWatch Billing Alarms
* SNS Notifications

These alerts help prevent unexpected charges caused by:

* Forgotten resources
* Misconfigured services
* Excessive traffic

---

# Cost Optimization Strategies

## Database Optimization

For non-critical workloads:

* Use Single-AZ RDS
* Save approximately 50% compared to Multi-AZ

---

## ECS Optimization

Only allocate the memory and CPU your application actually requires.

Recommended:

| Environment | Memory |
| ----------- | ------ |
| Test        | 512 MB |
| Production  | 1 GB   |

---

## RDS Reserved Instances

If the database will run continuously for 12+ months:

* Purchase a 1-year Reserved Instance
* Save approximately 30–40%

---

## AWS Budgets

Create monthly budgets to:

* Track spending
* Generate alerts
* Prevent budget overruns

---

## ECR Lifecycle Policies

Automatically delete old images:

```text
Keep Last 10 Images
Delete Older Versions
```

Benefits:

* Reduced storage costs
* Cleaner repositories

---

## CloudWatch Log Retention

Default retention:

```text
Never Expire
```

Recommended retention:

```text
30 Days
```

Benefits:

* Lower storage costs
* Reduced long-term log accumulation

---

# Monthly Cost Comparison Summary

| Service                   | Test Deployment | Production (Low Traffic) | Production (Medium Traffic) |
| ------------------------- | --------------- | ------------------------ | --------------------------- |
| RDS MySQL                 | $0.00           | $23.36                   | $23.36                      |
| ECS Fargate               | $8.50           | $35.74                   | $35.74                      |
| Application Load Balancer | $16.40          | $18.40                   | $24.40                      |
| S3 Frontend Hosting       | $0.00           | $0.00                    | $0.00                       |
| CloudFront                | $0.00           | $0.00                    | $8.50                       |
| Amazon ECR                | $0.05           | $0.05                    | $0.05                       |
| CodePipeline              | $0.00–1.00      | $1.00                    | $1.00                       |
| CodeBuild                 | $0.00           | $0.00                    | $0.00                       |
| Secrets Manager           | $0.00           | $2.00                    | $2.00                       |
| Route 53                  | $0.00           | $0.90                    | $2.50                       |
| AWS WAF                   | $0.00           | $5.00                    | $8.00                       |
| CloudWatch Logs           | $0.00           | $2.50                    | $10.00                      |
| ACM                       | $0.00           | $0.00                    | $0.00                       |
| Data Transfer             | $0.00           | $0.00                    | $0.00                       |

---

# Estimated Monthly Totals

| Environment                 | Estimated Monthly Cost |
| --------------------------- | ---------------------- |
| Test Deployment             | ~$25.00                |
| Production (Low Traffic)    | ~$87.00                |
| Production (Medium Traffic) | ~$135.00               |

---

# Disclaimer

All pricing figures are estimates based on standard AWS on-demand pricing in the **US East (N. Virginia)** region as of **June 2026**.

Actual costs may vary based on:

* AWS region
* Traffic volume
* Storage consumption
* Build frequency
* Logging volume
* Free tier eligibility

For the most accurate estimate, use the AWS Pricing Calculator and enter your exact deployment configuration.
