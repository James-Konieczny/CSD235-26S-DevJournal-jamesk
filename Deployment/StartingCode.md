# Starting Code - Deployment Requirements
**Purpose:**
- Read all the files in the OHS Remote project that can guide you on what it shall or shall not be changed in the project for deployment

- Guide for how to get code ready for launch

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