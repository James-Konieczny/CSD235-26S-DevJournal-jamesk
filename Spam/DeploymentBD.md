# Deployment Brain Dump
**Goal:**

I am responsible for the research on how to deploy the project.
1. Read all the files in the project that can guide you on what it shall or not be changed in the project for the development
2. Search online for examples on how to do this deployment
3. Search for good and safe domain options to deploy the application
4. Present the options and alterations to the Group Leader and Sponsor

- Private server (test deployment) for Jenifer viewing
- Get OHS Remote up on a server for public access (offical launch)

**Pieces:**
- Backend – the “engine” (FastAPI Python code)
- Frontend – the “dashboard and steering wheel” (React app)
- Database – the “memory” (MySQL)
- Environment variables (secrets like API keys)

## Steps to consider

1. Rent a computer in the cloud

Sign up for a service like Render, Railway, Heroku, AWS, or Azure.
They give you a virtual machine (or container) with a public IP address. Many have free tiers good enough for a test deployment.

2. Set up a database in the cloud

Instead of using MySQL inside a local Docker container, you create a managed MySQL database from the same cloud provider. They give you a connection string (looks like mysql://username:password@host:port/dbname).

3. Put your backend on the cloud computer

You package your backend into a Docker container (the previous team already has a Dockerfile).

You upload that container to a container registry (like Docker Hub) or directly deploy it on a platform that understands Docker.

You tell the cloud computer to run your container, passing in all the environment variables (Stripe keys, OpenAI key, database URL, etc.) as secrets – never hardcoded.

4. Put your frontend on a static hosting service

Your React frontend is just static files (HTML, CSS, JS) after running npm run build.
You can upload the dist/ folder to a service like Netlify, Vercel, or even the same cloud provider’s static storage (e.g., AWS S3 + CloudFront).
These services give you a public URL (e.g., https://your-app.netlify.app).

5. Connect frontend to backend

You change the frontend’s API base URL (currently hardcoded to http://localhost:8000) to the public URL of your deployed backend (e.g., https://your-backend.onrender.com/api/v1).
Because the frontend runs in a browser, that backend URL must be accessible from the internet and must have CORS configured to allow requests from the frontend’s URL.

6. Update Stripe webhook

Stripe needs to call your backend when a payment succeeds. In the Stripe Dashboard, you change the webhook endpoint URL to your backend’s public URL + /api/v1/payments/webhook.
You also make sure the webhook secret in your environment variables matches.

7. (Optional) Get a domain name and SSL

Platforms usually give you a something.onrender.com or something.netlify.app URL with automatic HTTPS (SSL). That’s fine for a test deployment. For a real product, you’d buy a domain like ohsremote.com and point it to your servers.

## How this applies
The previous team left a docker-compose.yml for local development only.

1. Choose a platform (Render.com is very beginner‑friendly – it supports both Docker containers for the backend and static sites for the frontend, plus a managed PostgreSQL – but you use MySQL. Railway.app is another good choice).

2. Prepare a production‑ready Dockerfile (the existing one likely works with small tweaks).

3. Set up a cloud MySQL database (e.g., Aiven, ClearDB, or the platform’s own MySQL service).

4. Deploy the backend as a web service.

5. Deploy the frontend as a static site.

6. Configure environment variables on the platform (all the STRIPE_*, OPENAI_API_KEY, SMTP_*, SECRET_KEY, DATABASE_URL).

7. Update the frontend’s API URL (you can either hardcode it temporarily for the test deployment or, as recommended, add a Vite environment variable like VITE_API_URL).

8. Update the Stripe webhook URL in the Stripe Dashboard.

9. Test the whole flow end‑to‑end on the live URLs.

## Short plan for a test deployment (Jennifer just needs to see it)
1. Sign up for Render.com (free tier).
2. Create a new Web Service – point it to your GitHub repo (Capstone347/ohs_backend).
3. Render will automatically detect the Dockerfile.
4. Add all environment variables from your .env.docker file.
5. Change DATABASE_URL to point to a Render MySQL database (they offer it as an add‑on).
6. Create a Static Site on Render – point it to your frontend repo, set build command npm run build, publish directory dist.
7. Under “Environment Variables”, add VITE_API_URL = the URL of your deployed backend.
8. Rebuild the frontend so it picks up that variable.
9. Update the Stripe webhook URL in the Stripe Dashboard to https://your-backend.onrender.com/api/v1/payments/webhook.
10. Give Jennifer the frontend URL (e.g., https://ohs-frontend.onrender.com).


------
## Info dump -- Tech Propsoal 

Plan A: Hybrid Plan (Student-Friendly/Easy)
Goal: Minimize complexity and maintenance overhead. We'll let specialized services handle what they do best while keeping the backend on AWS for database access.

Strategy:

Frontend: Netlify (GitHub integration, automatic builds, CDN)

Backend: AWS Elastic Beanstalk (Docker platform) + RDS

CI/CD: GitHub Actions pushes to Netlify (frontend) and Elastic Beanstalk (backend)

📁 Phase 1: Code Preparation
Frontend (ohs_frontend):

Create ./Dockerfile for build stage (optional; Netlify can build directly)

Configure vite.config.ts with base path if needed

Set environment variables in Netlify UI later (not in code)

Backend (ohs_backend):

Ensure ./Dockerfile exists and is production-ready

Update docker-compose.yml to separate services for local dev vs. production

Create ./entrypoint.sh or similar to run Alembic migrations before starting the app

Ensure all secrets are read from environment variables (not hardcoded)

🔧 Phase 2: AWS Setup for Backend
2.1 RDS (MySQL) - Free Tier
Steps:

Open AWS Console → RDS → Create database

Choose Standard Create → MySQL → Free tier template

Set DB instance identifier: ohs-db

Set master username/password (store securely)

Under Connectivity:

VPC: Default VPC

Public accessibility: No (keep private)

VPC security group: Create new or use default

Availability Zone: No preference

Under Additional configuration:

Initial database name: ohsdb

Backup retention period: 7 days (free tier allows)

Monitoring: Disable Enhanced Monitoring to avoid charges

Click Create database

Once created, note the Endpoint (hostname)

Security Group Setup:

Go to EC2 → Security Groups → Find the RDS security group

Add Inbound rule: Type MySQL/Aurora (port 3306), Source = sg-xxxxxxxx (Elastic Beanstalk security group)

This allows only your backend to connect, not the public internet

2.2 Elastic Beanstalk (FastAPI Docker) - Free Tier Eligible
Steps:

Open AWS Console → Elastic Beanstalk → Create application

Application name: ohs-backend

Environment → Web server environment

Platform: Docker (select "Docker" from the list)

Platform branch: Docker running on 64bit Amazon Linux 2

Application code: Upload your code (zip of your repo) or connect to GitHub

Presets: Free tier eligible configuration

Under Configure service access:

Create or use existing IAM role: aws-elasticbeanstalk-ec2-role

Ensure role has permissions for S3, CloudWatch, and RDS

Under Configure updates, monitoring, and logging:

Set Health reporting to basic

Additional configuration → Environment properties:

DATABASE_URL: mysql+pymysql://username:password@endpoint:3306/ohsdb

STRIPE_SECRET_KEY: your Stripe secret

OPENAI_API_KEY: your OpenAI key

CORS_ORIGINS: your Netlify frontend URL

SECRET_KEY: a strong random string for sessions

Click Create environment

Docker-specific configuration:

Elastic Beanstalk will automatically build your Dockerfile and run it

If you have a Dockerrun.aws.json file, it will use that instead

To pre-build images for faster deployment, push to ECR and reference in Dockerrun.aws.json

🖥️ Phase 3: Frontend Hosting
Netlify Setup (Free)
Steps:

Go to netlify.com → Sign up with GitHub

Click Add new site → Import an existing project

Select your ohs_frontend repository

Build settings:

Base directory: (leave empty if your package.json is at root)

Build command: npm run build

Publish directory: dist

Environment variables (add these):

VITE_API_BASE_URL: your Elastic Beanstalk environment URL

Click Deploy site

Important: Netlify's free tier includes:

100 GB/month bandwidth

300 build minutes/month

Unlimited team members

Instant cache invalidation

Automatic HTTPS and CDN

⚙️ Phase 4: CI/CD Pipeline (GitHub Actions)
Create .github/workflows/deploy.yml in both repositories.

Backend CI/CD (Push to Elastic Beanstalk)
yaml
name: Deploy Backend to AWS

on:
  push:
    branches: [ main ]
    paths:
      - 'ohs_backend/**'

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1
      
      - name: Zip and upload to S3
        run: |
          zip -r backend.zip ./ohs_backend -x "*.git*" "__pycache__*" "*.pyc"
          aws s3 cp backend.zip s3://elasticbeanstalk-us-east-1-account-id/backend.zip
      
      - name: Deploy to Elastic Beanstalk
        run: |
          aws elasticbeanstalk create-application-version \
            --application-name ohs-backend \
            --version-label ${{ github.sha }} \
            --source-bundle S3Bucket="elasticbeanstalk-us-east-1-account-id",S3Key="backend.zip"
          aws elasticbeanstalk update-environment \
            --environment-name ohs-backend-env \
            --version-label ${{ github.sha }}
Environment secrets needed in GitHub:

AWS_ACCESS_KEY_ID

AWS_SECRET_ACCESS_KEY

RDS_DATABASE_URL

STRIPE_SECRET_KEY

OPENAI_API_KEY

CORS_ORIGINS

SECRET_KEY

Frontend CI/CD (Push to Netlify)
yaml
name: Deploy Frontend to Netlify

on:
  push:
    branches: [ main ]
    paths:
      - 'ohs_frontend/**'

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Install dependencies
        run: npm ci
        working-directory: ./ohs_frontend
      
      - name: Build
        run: npm run build
        working-directory: ./ohs_frontend
        env:
          VITE_API_BASE_URL: ${{ secrets.VITE_API_BASE_URL }}
      
      - name: Deploy to Netlify
        uses: nwtgck/actions-netlify@v3.0
        with:
          publish-dir: './ohs_frontend/dist'
          production-branch: main
          github-token: ${{ secrets.GITHUB_TOKEN }}
          deploy-message: "Deploy from GitHub Actions"
        env:
          NETLIFY_AUTH_TOKEN: ${{ secrets.NETLIFY_AUTH_TOKEN }}
          NETLIFY_SITE_ID: ${{ secrets.NETLIFY_SITE_ID }}
🗄️ Phase 5: Database Migrations (Alembic)
Since your backend uses Alembic, you need a way to run migrations on the production RDS database.

Option 1 (Recommended for simplicity): Create a one-off GitHub Action workflow

yaml
name: Run Alembic Migrations

on:
  workflow_dispatch:  # Manual trigger only

jobs:
  migrate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'
      
      - name: Install dependencies
        run: pip install -r requirements.txt
      
      - name: Run Alembic migrations
        env:
          DATABASE_URL: ${{ secrets.RDS_DATABASE_URL }}
        run: alembic upgrade head
Option 2 (Automatic): Run migrations inside your Docker container before starting the app

Create entrypoint.sh:

bash
#!/bin/bash
alembic upgrade head
exec uvicorn main:app --host 0.0.0.0 --port 8000
Then in your Dockerfile:

dockerfile
COPY entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]
Important: Never auto-run migrations on production without proper testing and rollback plans. Option 1 (manual) is safer for student projects.

💰 Estimated Monthly Cost (Hybrid Plan)
Service	Cost
Netlify	$0 (free tier)
AWS RDS (db.t3.micro, 20GB storage)	$0 (free tier for 12 months)
AWS Elastic Beanstalk (t2.micro instance)	$0 (free tier for 12 months)
AWS Data Transfer (within free tier limits)	$0 (first 100GB/month across all services)
Total:	$0/month for first 12 months
🗄️ Plan B: AWS-Only Plan (One Provider/Client-Focused)
Goal: All services under one AWS account, easier for billing, IAM, and client management.

🔧 Service Mapping
Component	AWS Service
Frontend hosting	S3 + CloudFront
Backend compute	Elastic Beanstalk (Docker)
Database	RDS (MySQL)
CI/CD	GitHub Actions + CodeBuild + CodePipeline
Container registry	ECR
Secrets	Parameter Store (or Secrets Manager)
DNS & SSL	Route 53 + ACM
Monitoring	CloudWatch
🏗️ Phase 1: Infrastructure Setup
1.1 S3 Bucket for Frontend (Static Hosting)
Steps:

Go to S3 → Create bucket → Name: ohs-frontend-bucket

Disable Block all public access (because CloudFront will be the origin, not direct public access)

Under Properties → Static website hosting → Enable → Index document: index.html, Error document: index.html (for SPA routing)

Set bucket policy to allow CloudFront to read objects (not public access)

Bucket policy:

json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Service": "cloudfront.amazonaws.com"
            },
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::ohs-frontend-bucket/*",
            "Condition": {
                "StringEquals": {
                    "AWS:SourceArn": "arn:aws:cloudfront::account-id:distribution/cloudfront-id"
                }
            }
        }
    ]
}
1.2 CloudFront CDN
Steps:

Go to CloudFront → Create distribution

Origin domain: your S3 bucket endpoint

Origin access: Origin Access Control (OAC) → Create new OAC (S3 will only allow CloudFront to read)

Viewer protocol policy: Redirect HTTP to HTTPS

Alternate domain names (CNAMEs): Add custom domain if you have one (e.g., app.ohs.com)

Custom SSL certificate: Request from ACM (free)

Default root object: index.html

Error pages: Create a custom error response for 403/404 to serve index.html (for client-side routing)

Create distribution

Free tier: CloudFront includes 1 TB data transfer and 10 million HTTP/S requests per month for free

1.3 ECR (Container Registry)
Steps:

Go to ECR → Create repository

Name: ohs-backend-repo

Visibility: Private

Tag immutability: Enabled (recommended)

IAM policy for GitHub Actions to push to ECR:

json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ecr:GetAuthorizationToken",
                "ecr:BatchCheckLayerAvailability",
                "ecr:GetDownloadUrlForLayer",
                "ecr:GetRepositoryPolicy",
                "ecr:DescribeRepositories",
                "ecr:ListImages",
                "ecr:DescribeImages",
                "ecr:BatchGetImage",
                "ecr:InitiateLayerUpload",
                "ecr:UploadLayerPart",
                "ecr:CompleteLayerUpload",
                "ecr:PutImage"
            ],
            "Resource": "*"
        }
    ]
}
1.4 RDS MySQL (same as Plan A)
Follow the same RDS setup as Plan A.

1.5 Parameter Store (Secrets Management)
Instead of storing secrets in environment variables, store them in AWS Systems Manager Parameter Store for better security.

Steps:

Go to Systems Manager → Parameter Store → Create parameter

For each secret, create a SecureString parameter:

/ohs/prod/DATABASE_URL

/ohs/prod/STRIPE_SECRET_KEY

/ohs/prod/OPENAI_API_KEY

/ohs/prod/SECRET_KEY

/ohs/prod/CORS_ORIGINS

IAM policy for Elastic Beanstalk to read parameters:

json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ssm:GetParameter",
                "ssm:GetParameters",
                "ssm:GetParametersByPath"
            ],
            "Resource": "arn:aws:ssm:us-east-1:account-id:parameter/ohs/prod/*"
        }
    ]
}
1.6 Elastic Beanstalk with ECR
Steps:

Go to Elastic Beanstalk → Create application → ohs-backend

Platform: Docker

Platform branch: Docker running on 64bit Amazon Linux 2

Environment type: Load-balanced (production) or Single instance (cheaper)

Under Configure service access → Instance profile: Ensure role has permissions for ECR pull (see IAM section below)

Under Configure updates, monitoring, and logging:

Set Health reporting to basic (free)

Disable enhanced monitoring

Under Software → Environment properties: Instead of adding secrets here, modify the Dockerfile to fetch from Parameter Store at runtime (or use a script)

Instance IAM role policy for ECR:

json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ecr:GetAuthorizationToken",
                "ecr:BatchCheckLayerAvailability",
                "ecr:GetDownloadUrlForLayer",
                "ecr:GetRepositoryPolicy",
                "ecr:DescribeRepositories",
                "ecr:ListImages",
                "ecr:BatchGetImage"
            ],
            "Resource": "*"
        }
    ]
}
Custom Dockerrun.aws.json version 2 (for multi-container or ECR images):

json
{
    "AWSEBDockerrunVersion": "2",
    "containerDefinitions": [
        {
            "name": "ohs-backend",
            "image": "account-id.dkr.ecr.us-east-1.amazonaws.com/ohs-backend-repo:latest",
            "essential": true,
            "memory": 512,
            "portMappings": [
                {
                    "hostPort": 8000,
                    "containerPort": 8000
                }
            ],
            "environment": [
                {
                    "name": "DATABASE_URL",
                    "value": "mysql+pymysql://user:pass@endpoint:3306/ohsdb"
                }
            ]
        }
    ]
}
⚙️ Phase 2: CI/CD Pipeline (Full AWS Integration)
Create .github/workflows/full-deploy.yml in the root of your monorepo (or per repository).

yaml
name: Full AWS Deployment

on:
  push:
    branches: [ main ]

env:
  AWS_REGION: us-east-1
  ECR_REPOSITORY: ohs-backend-repo
  ECR_REGISTRY: ${{ secrets.ECR_REGISTRY }}
  S3_BUCKET: ohs-frontend-bucket
  DISTRIBUTION_ID: ${{ secrets.CLOUDFRONT_DISTRIBUTION_ID }}

jobs:
  build-and-deploy-backend:
    name: Build Backend Image and Deploy
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          role-to-assume: arn:aws:iam::account-id:role/github-actions-role
          aws-region: ${{ env.AWS_REGION }}

      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v1

      - name: Build, tag, and push image to Amazon ECR
        id: build-image
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          IMAGE_TAG: ${{ github.sha }}
        run: |
          cd ohs_backend
          docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
          echo "image=$ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG" >> $GITHUB_OUTPUT

      - name: Deploy to Elastic Beanstalk
        run: |
          aws elasticbeanstalk update-environment \
            --environment-name ohs-backend-env \
            --version-label ${{ github.sha }}

  build-and-deploy-frontend:
    name: Build Frontend and Deploy to S3
    runs-on: ubuntu-latest
    needs: build-and-deploy-backend
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          role-to-assume: arn:aws:iam::account-id:role/github-actions-role
          aws-region: ${{ env.AWS_REGION }}

      - name: Use Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Install dependencies
        run: npm ci
        working-directory: ./ohs_frontend

      - name: Build with environment variables
        run: npm run build
        working-directory: ./ohs_frontend
        env:
          VITE_API_BASE_URL: ${{ secrets.VITE_API_BASE_URL }}

      - name: Sync to S3
        run: |
          aws s3 sync ./ohs_frontend/dist s3://$S3_BUCKET --delete

      - name: Invalidate CloudFront cache
        run: |
          aws cloudfront create-invalidation \
            --distribution-id $DISTRIBUTION_ID \
            --paths "/*"

  run-migrations:
    name: Run Alembic Migrations
    runs-on: ubuntu-latest
    needs: build-and-deploy-backend
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          role-to-assume: arn:aws:iam::account-id:role/github-actions-role
          aws-region: ${{ env.AWS_REGION }}

      - name: Retrieve secrets from Parameter Store
        run: |
          DATABASE_URL=$(aws ssm get-parameter --name /ohs/prod/DATABASE_URL --with-decryption --query Parameter.Value --output text)
          echo "DATABASE_URL=$DATABASE_URL" >> $GITHUB_ENV

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'

      - name: Install dependencies
        run: pip install -r requirements.txt
        working-directory: ./ohs_backend

      - name: Run migrations
        run: alembic upgrade head
        working-directory: ./ohs_backend
        env:
          DATABASE_URL: ${{ env.DATABASE_URL }}
Required GitHub secrets:

ECR_REGISTRY (e.g., account-id.dkr.ecr.us-east-1.amazonaws.com)

CLOUDFRONT_DISTRIBUTION_ID

VITE_API_BASE_URL (your Elastic Beanstalk backend URL)

IAM role ARN for GitHub Actions OIDC (instead of long-term access keys)

🔒 Phase 3: IAM Setup for Least-Privilege Access
Role for GitHub Actions (OIDC):

json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "sts:AssumeRoleWithWebIdentity",
            "Principal": {
                "Federated": "arn:aws:iam::account-id:oidc-provider/token.actions.githubusercontent.com"
            },
            "Condition": {
                "StringEquals": {
                    "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
                }
            }
        }
    ]
}
Attached policies:

ECR-Push-Policy (custom)

S3-Frontend-Sync-Policy (custom)

CloudFront-Invalidation-Policy (custom)

ElasticBeanstalk-Deploy-Policy (custom)

SSM-GetParameter-Policy (custom for migrations)

🌐 Phase 4: DNS & SSL (Custom Domain)
Route 53:

Register or transfer domain to Route 53 (or use existing nameservers)

Create hosted zone

Create A record (alias) pointing to CloudFront distribution

ACM (Certificate Manager):

Request public certificate for *.ohs.com and ohs.com

Validate via DNS (Route 53)

Attach certificate to CloudFront distribution

CloudFront custom domain:

Edit CloudFront distribution → Alternate domain names → Add your domain

Custom SSL certificate → Select ACM certificate

💰 Estimated Monthly Cost (AWS-Only)
Service	Cost
S3 (5GB storage, 20GB transfer)	$0 (free tier)
CloudFront (50GB transfer)	$0 (free tier)
ECR (500MB storage)	$0 (free tier)
RDS (db.t3.micro, 20GB)	$0 (free tier for 12 months)
Elastic Beanstalk (t2.micro)	$0 (free tier for 12 months)
Route 53 (hosted zone)	$0.50/month
ACM	$0
Parameter Store (standard tier)	$0 (first 10,000 parameters free)
CloudWatch (basic metrics)	$0 (included)
Total with custom domain:	$0.50/month after free tier expiration
Total without custom domain:	$0/month for first 12 months
📊 Comparison: Hybrid vs. AWS-Only
Aspect	Hybrid Plan	AWS-Only Plan
Cost	$0 for first year	$0.50/month with custom domain
Time to deploy	1-2 hours	4-6 hours
Client management	Separate logins (Netlify + AWS)	Single AWS account
Learning value	Learn two platforms	Deep AWS expertise
Maintenance	Lower (Netlify auto-HTTPS, auto-CDN)	Higher (must configure CloudFront, ACM yourself)
Frontend previews	Netlify auto creates PR previews	Must configure separately
Backend scaling	AWS Elastic Beanstalk	AWS Elastic Beanstalk (same)
Best for	Student portfolio, quick start	Resume/CV showcase, client projects
🧪 Testing Checklist Before Deployment
Docker build locally: docker build -t ohs-backend . && docker run -p 8000:8000 ohs-backend

Environment variables: All secrets removed from code, only read via os.getenv()

Alembic migration locally: alembic upgrade head against local MySQL

CORS configured: Backend accepts requests from frontend URL (both Netlify preview and production)

GitHub Actions secrets: All required secrets added to repository

Database migration script tested against test RDS instance

📚 Additional College-Level Resources
Practical AWS Deployment (for both plans):

*AWS Certified Developer – Associate (DVA-C02) Complete Study Guide* by David Clinton (Sybex, 2024) – Chapters 6-8 cover Elastic Beanstalk, S3, RDS, and CodePipeline in depth

Deploying Web Applications on AWS: A Step-by-Step Guide – Free resource from the AWS Educate portal (accessible with your .edu email)

Docker & Containerization:

Containerization with Docker by Gabriel N. Schenker (Packt, 2023) – Covers multi-stage builds, environment injection, and production best practices

GitHub Actions:

GitHub Actions Cookbook by Michael L. B. (Apress, 2024) – Practical recipes for AWS OIDC integration and secret management