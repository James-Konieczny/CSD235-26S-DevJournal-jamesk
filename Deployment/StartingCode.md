# Starting Code - Deployment Requirements
**Purpose:**
- Read all the files in the OHS Remote project that can guide you on what it shall or shall not be changed in the project for deployment

- Guide for how to get code ready for launch
- [Jump to Frontend Required Changes](#frontend-required-changes)

## Backend Required Changes

### Change 1 – Create a production‑ready Docker Compose file
**Files involved:**
- `docker-compose.yml` → copy to `docker-compose.prod.yml`

**Description:**
- The existing `docker-compose.yml` is made for local development. It runs a local MySQL container, an `ngrok` tunnel, and mounts your source code for live reload.
- For production, create a separate Compose file that:
    - Removes the `ngrok` service.
    - Removes the local `mysql` service (use a cloud database instead).
    - Removes volume mounts that replace the code inside the container.
    - Removes the `--reload` flag from `uvicorn`.
    - Adds a `restart: always` policy.

**Why this change is needed:**
In production,
- never run a database inside the same Docker Compose stack (it’s not durable or scalable)
- never use ngrok (server already has a public IP)
- never mount code folders (the image itself contains the code)
- never use auto‑reload (it wastes resources)

**Sub-steps**:
1. In the backend repository root, copy the existing file:
```bash
cp docker-compose.yml docker-compose.prod.yml
```
2. Open `docker-compose.prod.yml`
3. Delete the entire `ngrok` service (from `ngrok:` down to its `ports:`).
- 
  - The `ngrok` service is not needed because your cloud server has its own public URL.
4. Delete the entire `mysql` service (from `mysql:` down to its `volumes:`).
- 
  - Use a managed cloud database instead.
5. In the `app:` service:
- 
  - Remove the `command:` line that contains 
```bash
uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload.
```
- 
  - Replace it with:
```yaml
command: uvicorn app.main:app --host 0.0.0.0 --port 8000 --workers 4
```
- (No `--reload`, and `--workers 4` for better performance.)
  - Remove all `volumes:` lines under `app:` (they mount local code).
  - Change restart from "no" to always.
  - Remove `stdin_open: true` and `tty: true` (not needed in production).
6. Remove the `volumes:` section at the bottom of the file (it was only for MySQL data).
7. The final docker-compose.prod.yml should look similar to this (only the app service remains, plus the network):
```yaml
version: '3.8'
services:
  app:
    build: .
    command: uvicorn app.main:app --host 0.0.0.0 --port 8000 --workers 4
    ports:
      - "8000:8000"
    env_file:
      - .env.docker
    restart: always
    networks:
      - ohs-network
networks:
  ohs-network:
    driver: bridge
```
8. Save the file.
------

### Change 2 – Set production environment variables

**Files involved:**

* `.env.docker.example` → copy to `.env.docker`
* Or configure the same values directly in your hosting platform's environment variable settings

**Description:**

* The backend reads all configuration from environment variables defined in `app/config.py`.
* The default values are intended for development and must be changed before deployment.
* Production settings should:

  * Disable debugging features.
  * Use a cloud-hosted MySQL database.
  * Configure proper frontend and backend URLs.
  * Restrict CORS to approved origins.
  * Use a secure secret key.
  * Reduce logging verbosity.

**Why this change is needed:**
In production,

* `ENVIRONMENT=development` enables debugging features and verbose logging.
* `DEBUG=true` can expose sensitive information.
* `DATABASE_URL` must point to a cloud database instead of the local Docker MySQL container.
* `ALLOWED_ORIGINS` must include the public frontend URL.
* `APP_BASE_URL` is used when generating links in emails.
* `FRONTEND_URL` is used for redirects after payment.
* `LOG_LEVEL` should be set to `INFO` or `WARNING` to reduce unnecessary log output.

**FAQ**

**Q: Where do I put these variables if I use a hosting platform such as Render or Heroku?**

A: Most platforms provide an Environment Variables or Secrets section in their dashboard. You can place all values there without creating a `.env.docker` file.

**Q: What if I still want to use a `.env.docker` file?**

A: That is acceptable for simple deployments. Ensure that the file is never committed to GitHub. It should already be listed in `.gitignore`.

**Q: How do I generate a new `SECRET_KEY`?**

A: Run:

```bash
openssl rand -hex 32
```

Copy the generated value into `SECRET_KEY`.

**Sub-steps:**

1. Copy the example file:

```bash
cp .env.docker.example .env.docker
```

2. Open `.env.docker`.

3. Update the following values (replace the examples with your actual values):

```ini
# Production mode
ENVIRONMENT=production
DEBUG=false
LOG_LEVEL=INFO

# Database – use your cloud MySQL URL
DATABASE_URL=mysql+pymysql://username:password@your-cloud-db-host:3306/ohs_remote_prod

# Backend public URL (without trailing slash)
APP_BASE_URL=https://your-backend.onrender.com

# Frontend public URL (without trailing slash)
FRONTEND_URL=https://your-frontend.onrender.com

# CORS – allow your frontend and optionally your backend
ALLOWED_ORIGINS=https://your-frontend.onrender.com,https://your-backend.onrender.com

# Generate a new secret key (do NOT reuse the example)
SECRET_KEY=your_new_64_character_hex_string

# Leave Stripe, OpenAI, SMTP, and ngrok values as they are,
# but verify that they contain valid values.
```

4. If your cloud provider supplies a database connection string, copy it into `DATABASE_URL`.

5. Save the file.

6. Verify that `.env.docker` is not committed to Git.

---

### Change 3 – Ensure the backend builds without development dependencies

**Files involved:**

* `docker/Dockerfile` (typically no changes required)
* `requirements.txt`
* `requirements-dev.txt` (new file if needed)

**Description:**

* The existing Dockerfile already uses a multi-stage build process.
* Verify that `requirements.txt` contains only runtime dependencies.
* Development tools should be moved into a separate `requirements-dev.txt` file.

**Why this change is needed:**
Production containers should contain only the packages required to run the application.

Benefits include:

* Smaller container images.
* Reduced attack surface.
* Faster startup times.
* Cleaner dependency management.

**FAQ**

**Q: Does the current `requirements.txt` contain development packages?**

A: The project documentation does not indicate that dependencies have been separated. Review the file manually.

**Q: Which packages are considered development-only?**

A: Typical examples include:

* `pytest`
* `ruff`
* `mypy`
* `black`
* `ipython`
* `ipdb`
* `pre-commit`

**Sub-steps:**

1. Open `requirements.txt`.

2. Identify packages used only for:

   * Testing
   * Linting
   * Formatting
   * Type checking

3. Create a new file:

```text
requirements-dev.txt
```

4. Move development-only packages into the new file.

5. Verify that the Dockerfile installs only production dependencies:

```dockerfile
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
```

6. Optionally use stricter installation behavior:

```dockerfile
RUN pip install --no-cache-dir --no-deps -r requirements.txt
```

7. Rebuild the image to confirm that the application still works:

```bash
docker build -f docker/Dockerfile -t ohs-backend:prod .
```

---

### Change 4 – Make sure database migrations run safely in production

**Files involved:**

* `docker-compose.prod.yml`
* Optional: `entrypoint.sh`

**Description:**

* In development, the application runs `alembic upgrade head` before starting Uvicorn.
* In production, migrations can either:

  * Run automatically during startup.
  * Be executed as a separate deployment step.

**Why this change is needed:**
When multiple backend instances start simultaneously, they may all attempt to run database migrations.

Although Alembic handles locking, running migrations as a dedicated deployment step is generally safer.

**FAQ**

**Q: Can I keep migrations in the startup command?**

A: Yes. For a single backend instance, this approach is simple and generally safe.

**Q: How do I run migrations manually?**

A: Execute:

```bash
docker exec -it <container_name> alembic upgrade head
```

**Sub-steps:**

1. Because `docker-compose.prod.yml` currently starts Uvicorn directly, choose one of the following options.

#### Option A – Simple approach (recommended)

2. Update the command in `docker-compose.prod.yml`:

```yaml
command: sh -c "alembic upgrade head && uvicorn app.main:app --host 0.0.0.0 --port 8000 --workers 4"
```

3. This will:

   * Apply migrations.
   * Start the application.

4. This approach is suitable for a single production instance.

#### Option B – Separate migration step

2. Remove migration execution from the startup command.

3. Deploy the application.

4. Run migrations manually:

```bash
docker exec -it $(docker ps -q -f name=app) alembic upgrade head
```

5. Restart the application container.

**Recommendation:**
Use **Option A** for test deployments and small production environments.

---

### Change 5 – Remove ngrok configuration (already completed)

**Files involved:**

* `docker-compose.prod.yml`
* `.env.docker`

**Description:**

* The `ngrok` service has already been removed.
* The `NGROK_AUTHTOKEN` environment variable may remain in `.env.docker`.

**Why this change is needed:**
No additional change is required.

The variable is simply ignored because ngrok is no longer used.

**Sub-steps:**

1. No action required.

---

### Change 6 – (Optional) Add a health check for the container platform

**Files involved:**

* `docker-compose.prod.yml`
* Optional: `docker/Dockerfile`

**Description:**

* Most cloud platforms use health checks to determine whether a container is functioning correctly.
* The backend already exposes:

```text
/api/v1/health
```

* Configure a health check so the platform can automatically restart unhealthy containers.

**Why this change is needed:**
Benefits include:

* Automatic recovery from failures.
* Better uptime.
* Improved deployment reliability.

**Sub-steps:**

1. Under the `app:` service in `docker-compose.prod.yml`, add:

```yaml
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost:8000/api/v1/health"]
  interval: 30s
  timeout: 10s
  retries: 3
  start_period: 40s
```

2. If the image does not contain `curl`, update the Dockerfile:

```dockerfile
RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*
```

3. Place this command after the existing system dependency installation section.

---

## Final Checklist for Backend Deployment Readiness

| Item                                                                                      | Done |
| ----------------------------------------------------------------------------------------- | ---- |
| Created `docker-compose.prod.yml` without ngrok, local MySQL, or live reload.             | ☐    |
| Removed volume mounts and added `restart: always`.                                        | ☐    |
| Set `ENVIRONMENT=production` and `DEBUG=false`.                                           | ☐    |
| Set `ALLOWED_ORIGINS` to include the production frontend URL.                             | ☐    |
| Set `DATABASE_URL` to the cloud MySQL instance.                                           | ☐    |
| Set `APP_BASE_URL` and `FRONTEND_URL` to public URLs.                                     | ☐    |
| Generated a new `SECRET_KEY` and stored it securely.                                      | ☐    |
| Verified `requirements.txt` contains only production dependencies.                        | ☐    |
| Added a health check (recommended).                                                       | ☐    |
| Tested the production image locally using: `docker-compose -f docker-compose.prod.yml up` | ☐    |

Once these changes are completed, the backend is ready to be deployed to Docker-compatible cloud platforms such as Render, Railway, DigitalOcean App Platform, AWS ECS, and similar services.


## Frontend Required Changes