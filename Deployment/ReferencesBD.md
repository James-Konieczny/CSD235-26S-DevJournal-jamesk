# 4. References & Sources 

## 4.1. Official AWS Documentation 

The following AWS documentation was consulted for the infrastructure design and deployment steps in Proposal 1. 

General AWS Architecture & Guidance 

What Is Cloud Computing? — AWS’s introduction to cloud fundamentals and service models. 
https://docs.aws.amazon.com/whitepapers/latest/aws-overview/what-is-cloud-computing.html 

Amazon ECS & AWS Fargate 

Amazon Elastic Container Service (ECS) Developer Guide — Comprehensive documentation covering clusters, task definitions, services, and Fargate launch type. 
https://docs.aws.amazon.com/AmazonECS/latest/developerguide/Welcome.html 

AWS Fargate — Serverless compute engine for containers; eliminates server management. 
https://docs.aws.amazon.com/AmazonECS/latest/developerguide/AWS_Fargate.html 

Fargate Task Networking — How to configure private subnets, security groups, and task networking. 
https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task-networking.html 

AWS CodePipeline (CI/CD) 

What Is AWS CodePipeline? — User guide for modelling, visualising, and automating release processes. 
https://docs.aws.amazon.com/codepipeline/latest/userguide/welcome.html 

Tutorial: Create a Four‑Stage Pipeline — Step‑by‑step walkthrough for a complete pipeline. 
https://docs.aws.amazon.com/codepipeline/latest/userguide/tutorials-four-stage-pipeline.html 

CodePipeline with GitHub (Version 2) — Connecting GitHub repositories as source actions. 
https://docs.aws.amazon.com/codepipeline/latest/userguide/connections-github.html 

Amazon RDS for MySQL 

Amazon Relational Database Service (RDS) User Guide — Managed MySQL, backup, restore, scaling, and Multi‑AZ. 
https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Welcome.html 

Working with MySQL DB Instances — Creation, configuration, and security for MySQL. 
https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_MySQL.html 

Multi‑AZ Deployments — High availability and automatic failover for production databases. 
https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.MultiAZ.html 

Amazon CloudFront (CDN) 

Amazon CloudFront Developer Guide — Content delivery network, caching, and custom domain configuration. 
https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/Introduction.html 

Using CloudFront with Amazon S3 Origins — Best practices for securing S3 origins with Origin Access Control. 
https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/DownloadDistS3AndCustomOrigins.html 

AWS WAF (Web Application Firewall) 

AWS WAF Developer Guide — Security rules, managed rule groups, Web ACLs, and protection against SQL injection and XSS. 
https://docs.aws.amazon.com/waf/latest/developerguide/waf-chapter.html 

AWS Managed Rule Groups — Pre‑configured protection sets (CommonRuleSet, IP Reputation, Known Bad Inputs). 
https://docs.aws.amazon.com/waf/latest/developerguide/aws-managed-rule-groups.html 

AWS Certificate Manager (ACM) 

AWS Certificate Manager User Guide — Requesting, validating, and renewing public SSL/TLS certificates for CloudFront and ALB. 
https://docs.aws.amazon.com/acm/latest/userguide/acm-overview.html 

AWS Secrets Manager 

What Is AWS Secrets Manager? — Storing and rotating database credentials, API keys, and other secrets. 
https://docs.aws.amazon.com/secretsmanager/latest/userguide/intro.html 

Best Practices for Managing Secrets — Encryption, access auditing, and rotation strategies. 
https://docs.aws.amazon.com/secretsmanager/latest/userguide/best-practices.html 

Amazon Route 53 (DNS) 

Amazon Route 53 Developer Guide — Domain registration, DNS routing, health checks, and alias records. 
https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/Welcome.html 

Amazon CloudWatch (Monitoring) 

Amazon CloudWatch User Guide — Metrics, logs, dashboards, and alarms for resource monitoring. 
https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/WhatIsCloudWatch.html 

Creating a Billing Alarm — Setting up alarms to monitor estimated AWS charges. 
https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/monitor_estimated_charges_with_cloudwatch.html 

## 4.2. Third‑Party Platform Documentation (Render, Fly.io, Railway, Supabase) 

The following official documentation and pricing pages were referenced for Proposal 2 (Cheaper Options). 

Render 

Render Documentation — Deploying web services, static sites, background workers, and cron jobs. Includes Docker support and managed TLS. 
https://render.com/docs 

Render Pricing Plans (April 2026 Update) — New workspace plans (Hobby, Pro, Scale) with flat pricing, no per‑seat fees. 
https://render.com/docs/plans 

Render Pricing FAQs — Billing details for custom domains ($0.25/domain/month) and outbound bandwidth ($0.15/GB beyond included allowance). 
https://render.com/docs/pricing 

Render Static Site Hosting — Deploy React apps directly from GitHub with automatic builds and HTTPS. 
https://render.com/docs/static-sites 

Fly.io 

Fly.io Documentation — Global application deployment with anycast routing, WireGuard tunnels, and automatic TLS certificates. 
https://fly.io/docs 

Fly.io Pricing — Usage‑based billing with a shared‑CPU machine (256 MB RAM) costing approximately $1.94/month and a 1 GB machine costing ~$5.70/month when always‑on. 
https://fly.io/docs/about/pricing 

Fly‑Postgres — Managed PostgreSQL on Fly.io with automated failover and backups. 
https://fly.io/docs/reference/postgres/ 

Railway 

Railway Documentation — All‑in‑one cloud platform with per‑second billing for compute and storage. 
https://docs.railway.com 

Railway Pricing Plans — Free trial ($5 one‑time credit, 30‑day expiry), Hobby ($5/month minimum), Pro ($20/month minimum), and Enterprise (custom). 
https://docs.railway.com/pricing 

Railway Resource Pricing — RAM: $10/GB/month, CPU: $20/vCPU/month, Network Egress: $0.05/GB, Volume Storage: $0.15/GB/month. 
https://docs.railway.com/pricing 

Railway Free Trial Details — 500 hours of compute, 512 MB RAM, 1 GB disk allocation during trial. 
https://docs.railway.com/free-trial 

Supabase 

Supabase Documentation — Open‑source Firebase alternative built on PostgreSQL. Includes Database, Auth, Storage, Realtime, and Edge Functions. 
https://supabase.com/docs 

Supabase Pricing — Free tier (500 MB database, 1 GB file storage, 2 GB egress, unlimited API requests) and Pro tier ($25/month) with increased limits and daily backups. 
https://supabase.com/pricing 

Supabase Database Migration Guides — Migrating from Render PostgreSQL, Heroku, AWS RDS, and other providers. 
https://supabase.com/docs/guides/migrations 

Supabase Row Level Security (RLS) — Security model for controlling database access at the row level. 
https://supabase.com/docs/guides/auth/row-level-security 

## 4.3. Relevant GitHub Repository Analysis 

The following repositories were directly examined in preparing the deployment requirements, especially the Required Code Changes section (Chapter 3). All URLs point to the official Capstone347 organisation on GitHub. 

Repository, URL, Key Findings 

ohs_frontend, https://github.com/Capstone347/ohs_frontend, React 18 / TypeScript / Vite application implementing a 5‑step wizard. Expects backend API at http://localhost:8000 during development. Includes comprehensive documentation: GETTING_STARTED.md, docs/ARCHITECTURE.md, docs/API_INTEGRATION.md, and docs/DEVELOPMENT_GUIDE.md. Admin dashboard available at /admin. 

ohs_backend, https://github.com/Capstone347/ohs_backend, FastAPI / Python 3.11 / MySQL / SQLAlchemy backend, fully Dockerised. Handles order lifecycle, Stripe payments, OpenAI SJP generation, and document production. docker-compose.yml defines the full development environment; Docker is the only supported development path. Uses Alembic for database migrations, Pydantic for configuration management, and includes a /health endpoint for readiness checks. 

The analysis of these repositories directly informed: 

The environment variable configuration (Section 3.1) — the frontend hard‑codes the backend URL, requiring replacement with VITE_API_BASE_URL. The backend already uses .env but needed full externalisation of secrets. 

The CORS and security headers (Section 3.2) — due to the frontend and backend running on separate origins in deployment, CORSMiddleware must be configured with the production frontend URL. 

The database migration automation (Section 3.3) — Alembic is already present; the entrypoint.sh script ensures migrations run automatically on container startup. 

Health checks and logging (Section 3.4) — the backend already has a /health endpoint; the proposal adds a database connectivity check and structured logging via Python’s logging module. 

Stripe and OpenAI key management (Section 3.5) — both repositories already reference these services; the proposal adds AWS Secrets Manager integration and key rotation best practices. 