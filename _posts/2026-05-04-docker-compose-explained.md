---
layout: post
title: "Stop Writing Docker Compose Files From Scratch — A Reusable Template for Any AI/ML Stack "
date: 2026-05-04
image: "docker-compose-portability.png"
description: "Reusable dockercompose.yml file for faster development"
---

# Stop Writing Docker Compose Files From Scratch — A Reusable Template for Any AI/ML Stack 🚀

> **Who is this for?** Anyone building an AI/ML project with multiple services — database, cache, vector store, API, background worker, and a dashboard. Understand this template once, swap the placeholders, and it works for every project you build.

---

## Why Docker Compose?

A real AI/ML project is rarely just one Python file. It typically needs:

- A **database** to store structured data
- A **vector database** to store and search embeddings
- A **cache** for fast, repeated lookups
- An **API** to handle requests
- A **background worker** for long-running tasks like ingestion
- A **dashboard** for users to interact with the system

Without Docker Compose, you'd manage six separate terminal windows, six different startup commands, and manually making sure each service is ready before the next one starts.

With Docker Compose, you run one command and everything comes up in the right order:

```bash
docker compose up -d --build
```

That's it. The full stack is running.

---

## How Hot Reload Works in Development

```
Your Machine                       Inside the Container
────────────────                   ────────────────────
/your-project/app/    ←─ bind ──→  /app/app/
/your-project/main.py ←─ mount──→  /app/main.py

You save a file
    ↓
The container sees the change immediately (shared filesystem)
    ↓
Uvicorn / Streamlit detects the change and restarts
    ↓
You refresh the browser — your new code is live ✅
```

You only need to rebuild the image when `requirements.txt` or the `Dockerfile` changes. For regular code edits, just save and refresh.

---

## The Full docker-compose.yml — With Every Line Explained

```yaml
# No need for a "version:" field in modern Docker Compose (v2+).
# Docker now follows the Compose Specification automatically.

services:

  # ══════════════════════════════════════════════
  # SERVICE 1: Relational Database
  # ══════════════════════════════════════════════
  postgres:
    image: postgres:16-alpine          # ← swap for your preferred DB version
    environment:
      - POSTGRES_DB=${POSTGRES_DB}             # pulled from your .env file
      - POSTGRES_USER=${POSTGRES_USER}
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
      # Named volume — Docker manages this storage location.
      # Without it, all your data is wiped every time the container is removed.
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER}"]
      interval: 5s    # check every 5 seconds
      timeout: 5s     # fail if no response within 5 seconds
      retries: 5      # declare "unhealthy" after 5 consecutive failures
    # ⚠️ No "ports:" here intentionally.
    # Postgres is only reachable by other containers, not from your host machine.
    # This is a deliberate security choice.


  # ══════════════════════════════════════════════
  # SERVICE 2: Vector Database
  # ══════════════════════════════════════════════
  qdrant:
    image: qdrant/qdrant:v1.12.0       # ← swap for Weaviate, Chroma, Milvus, etc.
    volumes:
      - ./qdrant_storage:/qdrant/storage
      # Bind mount — data is stored in a folder right next to your docker-compose.yml.
      # You can open that folder and inspect the files directly on your machine.
      # Different from named volumes which are hidden inside Docker's storage area.
    healthcheck:
      test: ["CMD", "bash", "-c", "cat < /dev/null > /dev/tcp/localhost/6333"]
      interval: 10s
      timeout: 5s
      retries: 5
    # ⚠️ No "ports:" — internal access only


  # ══════════════════════════════════════════════
  # SERVICE 3: Cache
  # ══════════════════════════════════════════════
  redis:
    image: redis:7-alpine
    volumes:
      - redis_data:/data    # Named volume — cache state survives container restarts
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 3s
      retries: 5
    # ⚠️ No "ports:" — internal access only


  # ══════════════════════════════════════════════
  # SERVICE 4: Main API
  # ══════════════════════════════════════════════
  api:
    build: .                     # Build using the Dockerfile in this same directory
    image: your-project:latest   # ← YOUR_PROJECT_NAME — this names the built image
    ports:
      - "8000:8000"
      # Format: "HOST_PORT:CONTAINER_PORT"
      # Left side  = your laptop    → http://localhost:8000
      # Right side = inside Docker  → container's port 8000

    # This overrides the CMD in your Dockerfile.
    # We add --reload here for development so the server restarts on file changes.
    command: uvicorn main:app --host 0.0.0.0 --port 8000 --reload

    volumes:
      # These bind mounts connect your local files to the container.
      # Edit your code locally → container sees it instantly → --reload restarts the server.
      - ./app:/app/app               # ← YOUR_APP_FOLDER
      - ./main.py:/app/main.py       # ← YOUR_ENTRYPOINT
      # Add more bind mounts for other folders you want live-reloaded

    env_file:
      - .env                   # Load all variables from your .env file

    environment:
      # These override or supplement what's in .env.
      # CRITICAL: containers talk to each other using SERVICE NAMES, not localhost.
      # "qdrant", "redis", "postgres" below are the service names defined above.
      - QDRANT_URL=http://qdrant:6333
      - REDIS_URL=redis://redis:6379/0
      - DATABASE_URL=postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@postgres:5432/${POSTGRES_DB}
      - TRUSTED_HOSTS=localhost,127.0.0.1,api,${TRUSTED_HOSTS}
      # TRUSTED_HOSTS prevents 400 errors when services call each other
      # using internal Docker hostnames like "api"

    depends_on:
      postgres:
        condition: service_healthy  # Wait until postgres passes its healthcheck
      redis:
        condition: service_healthy  # Wait until redis passes its healthcheck
      qdrant:
        condition: service_healthy  # Wait until qdrant passes its healthcheck
    # Without "condition: service_healthy", depends_on only controls startup ORDER,
    # not readiness. Your API could start before Postgres is accepting connections.

    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/docs"]
      # Hits the /docs endpoint (FastAPI auto-generates this).
      # Replace with any endpoint that returns 200 when your API is ready.
      interval: 10s
      timeout: 5s
      retries: 5


  # ══════════════════════════════════════════════
  # SERVICE 5: Background Worker
  # ══════════════════════════════════════════════
  worker:
    build: .
    image: your-project:latest   # Reuses the same image as the API — no rebuild needed
    volumes:
      - ./app:/app/app
    command: arq app.worker.WorkerSettings
    # ↑ Replace with your own worker command:
    #   Celery:  celery -A your_app.worker worker --loglevel=info
    #   RQ:      rq worker --url redis://redis:6379/0
    #   Arq:     arq your_app.worker.WorkerSettings
    env_file:
      - .env
    environment:
      - QDRANT_URL=http://qdrant:6333
      - REDIS_URL=redis://redis:6379/0
      - DATABASE_URL=postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@postgres:5432/${POSTGRES_DB}
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
      qdrant:
        condition: service_healthy
    # No healthcheck needed here — the worker doesn't expose a port


  # ══════════════════════════════════════════════
  # SERVICE 6: Frontend Dashboard
  # ══════════════════════════════════════════════
  dashboard:
    build: .
    image: your-project:latest
    command: streamlit run dashboard.py --server.port=8501 --server.address=0.0.0.0
    # ↑ Replace with your frontend command:
    #   Gradio:    python app.py
    #   Streamlit: streamlit run dashboard.py ...
    #   Next.js:   npm run dev (use a different image for Node-based frontends)
    ports:
      - "8501:8501"     # Access at http://localhost:8501
    volumes:
      - ./dashboard:/app/dashboard
      - ./dashboard.py:/app/dashboard.py
      - ./app:/app/app   # Shared logic — both API and Dashboard can use it
    environment:
      - API_BASE_URL=http://api:8000/api/v1
      # ↑ "api" is the service name, not localhost. This is a very common mistake.
    depends_on:
      api:
        condition: service_healthy  # Dashboard starts only after API is ready


# ══════════════════════════════════════════════
# VOLUMES — Persistent Storage Declaration
# ══════════════════════════════════════════════
volumes:
  postgres_data:   # Docker manages this — data survives container removal
  redis_data:      # Docker manages this — same
  # qdrant_storage is a bind mount (./qdrant_storage) so it's not declared here
```

---

## The Three Concepts That Trip Everyone Up

### 1. Port Mapping — Left is You, Right is the Container

```
"8000:8000"
  ↑      ↑
Your     Container's
machine  internal port

You access it at: http://localhost:8000
```

Postgres, Redis, and Qdrant have **no ports exposed** in this setup. They're only reachable by other containers on the same Docker network. If someone gains access to your host, they can't reach the database directly. That's intentional.

### 2. Service Names as Hostnames — Never Use localhost

This is the most common mistake when moving from single-service to multi-service Docker setups.

```
❌ WRONG:  http://localhost:8000    (this points to the container's own localhost)
✅ RIGHT:  http://api:8000          (this points to the "api" service)

❌ WRONG:  postgresql://localhost:5432
✅ RIGHT:  postgresql://postgres:5432

❌ WRONG:  redis://localhost:6379
✅ RIGHT:  redis://redis:6379
```

Docker Compose creates a private network. Every service's name automatically becomes its hostname on that network.

### 3. depends_on with healthcheck — Startup Order vs Readiness

```
Without "condition: service_healthy":

  postgres container starts
       ↓ (immediately, before postgres is ready to accept connections)
  api container starts
       ↓
  "Connection refused" error ❌

With "condition: service_healthy":

  postgres container starts
       ↓ (waits for pg_isready to pass...)
       ↓ (postgres is now actually accepting connections)
  api container starts ✅
```

The `depends_on` key alone only controls start order. Adding `condition: service_healthy` ensures the dependency is truly ready, not just started.

---

## Named Volumes vs Bind Mounts — When to Use Which

| | Named Volume | Bind Mount |
|---|---|---|
| **Syntax** | `postgres_data:/var/lib/...` | `./app:/app/app` |
| **Managed by** | Docker | You (it's a folder on your machine) |
| **Visible on host** | Not easily | Yes, directly |
| **Best for** | Database data, Redis persistence | Code hot reload during development |
| **On container removal** | Data survives | Data survives (it's on your machine) |

---

## Your .env File

```bash
# .env
# Add this file to .gitignore — never commit secrets to version control

# Database
POSTGRES_DB=your_database_name
POSTGRES_USER=your_db_user
POSTGRES_PASSWORD=a_strong_password_here

# Trusted hosts
TRUSTED_HOSTS=yourdomain.com

# API keys (add whatever your project needs)
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=sk-ant-...
# YOUR_OTHER_API_KEY=...
```

---

## Commands You'll Use Every Day

```bash
# Start the full stack (with image rebuild)
docker compose up -d --build

# Start the full stack (reuse existing images, just restart)
docker compose up -d

# Follow logs for a specific service
docker compose logs -f api
docker compose logs -f worker

# See the status of all services
docker compose ps

# Open a shell inside a running container
docker compose exec api bash
docker compose exec postgres psql -U your_db_user -d your_database_name

# Stop everything
docker compose down

# Stop everything AND delete all volumes (WARNING: this deletes your data)
docker compose down -v

# Restart a single service without touching the others
docker compose restart api
```

---

## What Changes Per Project vs What Never Changes

Just like the Dockerfile, the vast majority of this file is permanent boilerplate. Here's exactly what you touch for each new project — and what you never need to rewrite.

| Always stays the same — never touch | Changes per project — your only edits |
|---|---|
| Multi-service structure (db, cache, vector db, api, worker, dashboard) | Service image versions (`postgres:16`, `redis:7`, etc.) |
| `depends_on` with `condition: service_healthy` pattern | Your project image name |
| Healthcheck pattern for each service type | Port numbers |
| Named volume declarations | Your app folder names (bind mounts) |
| Internal URLs using service names (not localhost) | Worker startup command (Celery / Arq / RQ) |
| `env_file: - .env` pattern | Dashboard startup command (Streamlit / Gradio) |
| No ports on internal services (db, cache, vector db) | `API_BASE_URL` path and version prefix |
| Hot reload via bind mounts + `--reload` flag | Which services your project actually needs |

For most new projects, you're editing around 10–15 lines. The networking logic, startup ordering, healthchecks, and volume strategy are all solved once and reused forever.

---

## Placeholders — Replace These With Your Own Values

| Placeholder | What to put here |
|---|---|
| `your-project:latest` | Your project name, e.g. `my-ml-app:latest` |
| `YOUR_APP_FOLDER` | Your `app/`, `src/`, or package folder name |
| `YOUR_ENTRYPOINT` | `main.py`, `app.py`, etc. |
| `YOUR_WORKER_COMMAND` | Your Celery / Arq / RQ worker startup command |
| `YOUR_DASHBOARD_COMMAND` | Streamlit, Gradio, or any frontend run command |
| `8000`, `8501` | Your chosen ports |
| `http://api:8000/api/v1` | Your actual API base URL and version prefix |
| `qdrant/qdrant` | Swap for Weaviate, Chroma, Milvus, etc. |

---

## The Full Picture — What You're Building

```
Browser / Client
      │
      ▼
┌─────────────┐    ┌─────────────────┐
│  Dashboard  │───▶│       API       │
│   :8501     │    │      :8000      │
└─────────────┘    └────────┬────────┘
                            │
              ┌─────────────┼─────────────┐
              ▼             ▼             ▼
        ┌──────────┐ ┌──────────┐ ┌──────────┐
        │ Postgres │ │ Vector DB│ │  Redis   │
        │          │ │          │ │  Cache   │
        └──────────┘ └──────────┘ └──────────┘
              ▲
              │
        ┌──────────┐
        │  Worker  │
        │(Background│
        │  Tasks)  │
        └──────────┘

All services share one Docker network.
They talk to each other using service names as hostnames.
Only the API and Dashboard ports are published to your host machine.
```

---

## Setup Checklist

```
□ Create your .env file and add it to .gitignore
□ Replace all placeholders with your actual service and folder names
□ Check ports — make sure nothing on your machine is already using them
□ Choose named volumes for data that needs to persist
□ Use bind mounts for code folders you want to hot reload
□ Set depends_on with condition: service_healthy for correct startup order
□ Use service names (not localhost) in all internal URLs
□ Run: docker compose up -d --build
□ Run: docker compose ps  →  all services should show as "healthy"
□ Open http://localhost:YOUR_API_PORT/docs  →  API is up
□ Open http://localhost:YOUR_DASHBOARD_PORT  →  Dashboard is up
```

---

*With this template, you can set up a full multi-service Docker stack for any AI/ML project. Fill in the placeholders, add your .env, and the entire system comes up with a single command.*
