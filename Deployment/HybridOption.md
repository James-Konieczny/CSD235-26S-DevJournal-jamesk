# 2. Proposal 2: Cheaper Options – AWS Free Tier, Render, and Alternatives

If your primary constraint is minimising upfront and recurring costs, a range of platforms offer generous free allowances and more predictable low-cost billing than a full AWS-only stack. This section evaluates leading alternatives – Render, Fly.io, Railway, and Supabase – describes how to build a working test deployment using only free resources, outlines a hybrid production strategy that stays well under the full AWS budget, and concludes with a candid assessment of the limitations that come with cheap or free hosting.

---

## 2.1 Alternative Platform Comparison

Each alternative platform below is a managed Platform-as-a-Service (PaaS) that can deploy a containerised FastAPI backend and a static React frontend with minimal configuration. Because `ohs_backend` is already containerised, each service can run it directly from a Dockerfile or GitHub repository.

### Render
- 750 hours/month free web service (512 MB RAM)
- Unlimited static site hosting
- No credit card required
- Spins down after 15 minutes of inactivity (cold start delay 3–8 seconds)
- Free PostgreSQL expires after 30 days

**Best fit:** Test deployments, sponsor previews, static frontend hosting

---

### Fly.io
- 3 always-on shared-CPU VMs (256 MB RAM each)
- 3 GB persistent volume storage
- 160 GB outbound bandwidth/month
- Global deployment with no cold starts
- Requires Docker + configuration (`fly.toml`)
- New accounts: $5 trial credit only (no permanent free tier)

**Limitations:**
- Legacy users retain free tier; new users do not
- Self-managed PostgreSQL

**Best fit:** Global low-latency APIs, microservices

---

### Railway
- $5 one-time trial credit (30 days)
- Free tier: ~0.5 GB RAM, 1 vCPU
- Usage-based billing after trial
- $1/month credit on free plan
- Auto-detects projects (minimal config)

**Pricing model:**
- $10/GB RAM/month
- $20/vCPU/month
- $0.05/GB egress

**Best fit:** Prototyping, lightweight full-stack apps

---

### Supabase
- 500 MB PostgreSQL database
- 1 GB file storage
- 5 GB egress
- Unlimited API requests
- Free forever tier
- Pauses after 1 week inactivity

**Limitations:**
- Database only (no backend hosting)

**Best fit:** Managed PostgreSQL for any stack

---

### Summary Table

| Platform  | Free Tier Highlights | Key Limitations | Best Fit |
|----------|----------------------|-----------------|----------|
| Render   | 750h/month web service, 512MB RAM, static hosting | Cold starts, DB expires after 30 days | Test deployments, previews |
| Fly.io   | 3 always-on VMs, 3GB storage, 160GB bandwidth | No permanent free tier for new users | Global APIs, microservices |
| Railway  | $5 trial, low-resource free tier | Very limited free compute | Prototypes |
| Supabase | Free PostgreSQL (500MB), storage, API | No backend compute | Database layer |

---

## 2.2 Test Deployment Using Free Services Only

A fully functional zero-cost test deployment:

### Frontend (React)
- Hosted on **Render Static Site**
- Automatic build from GitHub
- Free SSL + CDN included
- Connects to backend API URL

---

### Backend (FastAPI)
- Render free web service (750 hours/month)
- Docker-based deployment from `ohs_backend`
- Spins down after 15 minutes inactivity
- Cold start delay: 3–8 seconds

**Optional mitigation:**
- Ping health endpoint periodically to reduce cold starts

---

### Database
- Use **Supabase Free PostgreSQL**
- 500 MB storage + 5 GB egress
- No expiry (pauses after inactivity)
- Suitable for development and demo use

---

### CI/CD
- Automatic deployment via GitHub push
- Render rebuilds backend + frontend automatically
- Live logs available in dashboard

---

### Important Notes
- Render free backend sleeps after inactivity → first request delayed
- Supabase pauses after 7 days inactivity → manual resume may be needed
- No production-grade guarantees on free tiers

---

## 2.3 Production Deployment on a Budget (Hybrid AWS + Render)

A cost-efficient production architecture:

### Frontend
- Render static site (free)
- Optional CloudFront for CDN (AWS cost optional)

---

### Backend
- Render Starter Plan: **$7/month**
  - Always-on service (no cold starts)
  - 512 MB RAM, 0.5 vCPU
- Optional worker service: $7/month

---

### Database
Option A:
- Supabase Pro: **$25/month**
  - 8 GB database storage
  - Automated backups
  - Point-in-time recovery

Option B:
- Render PostgreSQL: ~$7/month
  - Simpler single-platform setup

---

### CI/CD
- Fully automated via Render Git integration
- Rolling deployments on paid plans
- Zero manual pipeline required

---

### Domain & TLS
- Free SSL via Render
- Custom domain supported
- Optional AWS Route 53: $0.50/month

---

### Final Architecture

- Backend: Render Starter → $7/month
- Frontend: Render Static → $0
- Database: Supabase Pro → $25/month

**Total: ~$32/month**

---

### Cost Efficiency Note
Fly.io and Railway may appear cheaper at low usage, but:
- Compute scales quickly with RAM requirements
- Database and bandwidth costs increase significantly
- Less predictable than Render + Supabase combo

---

## 2.4 Cost Breakdown & Limitations

### Low-Traffic Production (Hybrid Model)

| Service | Configuration | Monthly Cost |
|--------|--------------|--------------|
| Render Web Service | Starter (512MB, always-on) | $7.00 |
| Render Static Site | Free hosting | $0.00 |
| Supabase Database | Pro plan | $25.00 |
| Domain & TLS | Included | $0.00 |

**Total: $32.00/month**

---

### Medium-Traffic Production (Hybrid Model)

| Service | Configuration | Monthly Cost |
|--------|--------------|--------------|
| Render Web Service | Standard (2GB RAM, 1 vCPU) | $25.00 |
| Render Static Site | Free | $0.00 |
| Supabase Database | Pro + scaling | $25–$65 |
| AWS Route 53 | Optional DNS | $0.50 |
| CloudFront | Optional CDN | $0–$8.50 |

**Total: $60–$75/month**

---

## Key Limitations of Cheaper Options

### Database Persistence Risk
- Render free PostgreSQL deletes after 30 days
- Requires migration to Supabase or paid DB for production

---

### Cold Starts
- Free services sleep after inactivity
- Causes 3–8 second delay on first request

---

### No Autoscaling
- Single container per service
- Performance degrades under high load

---

### Limited Regions
- Fewer deployment regions than AWS
- Higher latency for distant users

---

### Supabase Throttling
- Shared CPU on free tier
- Upgrade required for consistent production load

---

### Vendor Lock-In Considerations
- Docker backend reduces portability risk
- Database migration still required if switching providers

---

### No Built-in File Storage (Render)
- Requires external storage (e.g. Cloudflare R2, Backblaze B2)
- Additional integration needed for uploads