# 3. Required Code Changes for Deployment

Before deploying to test or production environments, several important changes must be made to both the `ohs_frontend` and `ohs_backend` codebases. These updates ensure the system is secure, environment-aware, and fully compatible with cloud deployment platforms.

The key goals are:
- Remove all hard-coded configuration
- Enable secure environment variable management
- Standardise database migrations
- Improve observability (health checks + logging)
- Strengthen security (CORS, headers, cookies)

---

## 3.1 Environment Variable Configuration (Frontend & Backend)

Hard-coded values are one of the most common causes of deployment failures. All environment-specific configuration must be externalised.

---

## Frontend (`ohs_frontend`)

The frontend currently uses a hard-coded API URL such as `http://localhost:8000`. This must be replaced with environment-based configuration using Vite.

### Step 1: Create environment files

- `.env.development`
- `.env.production`

**.env.development**
VITE_API_BASE_URL=http://localhost:8000

**.env.production**
VITE_API_BASE_URL=https://api.yourdomain.com

Replace with your deployed backend URL.

### Step 2: Update API client

const API_BASE_URL = import.meta.env.VITE_API_BASE_URL;

export const apiClient = axios.create({
  baseURL: API_BASE_URL,
  withCredentials: true,
});

### Step 3: Build script

"build": "vite build"

### Step 4: Remove hardcoded URLs

Search for localhost:8000 and 127.0.0.1:8000

---

## Backend (`ohs_backend`)

### Step 1: .env.example

DATABASE_URL=mysql://user:password@host:3306/ohsdb
SECRET_KEY=your-strong-secret-key
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30

STRIPE_SECRET_KEY=sk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...

OPENAI_API_KEY=sk-...

SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=email@example.com
SMTP_PASSWORD=app-password

FRONTEND_URL=https://yourdomain.com
ENVIRONMENT=development

### Step 2: Pydantic Settings

from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    database_url: str
    secret_key: str
    algorithm: str = "HS256"
    access_token_expire_minutes: int = 30

    stripe_secret_key: str
    stripe_webhook_secret: str
    openai_api_key: str

    smtp_host: str
    smtp_port: int = 587
    smtp_user: str
    smtp_password: str

    frontend_url: str
    environment: str = "development"

    class Config:
        env_file = ".env"

settings = Settings()

---

## 3.2 CORS & Security

from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=[settings.frontend_url],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

---

## 3.3 Database Migrations

entrypoint.sh:
#!/bin/sh
set -e
alembic upgrade head
exec uvicorn app.main:app --host 0.0.0.0 --port 8000

---

## 3.4 Health Check

@app.get("/health")
async def health_check():
    return {"status": "ok"}

---

## 3.5 Logging

import logging

logging.basicConfig(level=logging.INFO)

logger = logging.getLogger(__name__)

---

## 3.6 Secrets

Use environment variables only:
settings.stripe_secret_key
settings.openai_api_key

Never commit secrets to Git.
