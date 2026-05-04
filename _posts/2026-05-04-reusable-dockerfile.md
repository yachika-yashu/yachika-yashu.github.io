---
layout: post
title: "The Only Dockerfile You'll Ever Need for AI/ML Projects 🐳 "
date: 2026-05-04
image: "download.png"
description: "Multi-Stage Build with detailed instructions"
---


# The Only Dockerfile You'll Ever Need for AI/ML Projects 🐳

> **Who is this for?** Anyone who wants to stop writing a new Dockerfile for every project. This is a reusable template — understand it once, replace a handful of placeholders, and it works for any AI/ML project you build.

---

## Why Docker in the First Place?

Imagine your AI project runs perfectly on your machine. You share it with a teammate — nothing works on their end. "But it works on my machine!" — this is the exact problem Docker solves.

Docker creates a **box** that contains your entire project — code, dependencies, OS libraries — all packed together. This box runs identically on any machine, guaranteed.

---

## Multi-Stage Build — The Factory Analogy 🏭

This is the **most important concept** in this Dockerfile. Think of it like a factory with two rooms:

```
Stage 1 (Builder Room):   This is where things are MADE
                          Compilers, build tools, headers — all live here
                          But none of this goes into the final product

Stage 2 (Runtime Room):   Only the FINISHED PRODUCT lives here
                          Just what's needed to actually run the app
```

**Why does this matter?** The builder stage pulls in ~200MB of extra tools. If all of that ended up in your production image, it would be bloated, slow to deploy, and a bigger security target. Multi-stage solves this — only the compiled output gets copied over. The image stays lean.

---

## Stage 1 — Builder (Build Everything Here)

```dockerfile
# ─────────────────────────────────────────────
# STAGE 1 — builder
# ─────────────────────────────────────────────
FROM python:3.11-slim AS builder
# ↑ Replace with your Python version: 3.10-slim, 3.12-slim, etc.

# Two very important environment variables:
# PYTHONDONTWRITEBYTECODE → skip .pyc cache files (saves image size)
# PYTHONUNBUFFERED        → logs appear immediately, no internal buffering
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

# Install system tools needed only to BUILD your Python dependencies.
# These never reach the final image — that's the whole point of Stage 1.
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \   # gcc/g++/make — compiles C extensions like numpy, scipy
    libpq-dev \         # PostgreSQL headers — needed to compile psycopg2 from source
    curl \              # for network calls / debugging during build
    && rm -rf /var/lib/apt/lists/*
    # ↑ Always clean up apt cache at the end — keeps the layer smaller

# Create a virtual environment at /venv instead of installing into system Python.
# Why? So Stage 2 can grab the entire compiled dependency tree
# with a single COPY --from=builder command.
RUN python -m venv /venv
ENV PATH="/venv/bin:$PATH"

# uv is a much faster dependency resolver than plain pip.
# It also automatically resolves conflicting package versions.
RUN pip install --upgrade pip uv

# ⚡ THE MOST IMPORTANT CACHING TRICK IN THIS FILE
# Copy requirements.txt BEFORE copying your source code.
#
# Docker builds in layers. If a layer hasn't changed, Docker reuses
# its cached version and skips re-running it.
#
# By copying requirements.txt first:
# → Code-only changes reuse the pip install cache (saves minutes per build)
# → Only an actual requirements.txt change triggers a fresh install
COPY requirements.txt /tmp/requirements.txt
RUN uv pip install --no-cache-dir -r /tmp/requirements.txt
```

**Visualizing the caching logic:**

```
[Copy requirements.txt]  ← Rebuilds only when requirements.txt changes
[uv pip install      ]   ← Rebuilds only when requirements.txt changes
[Copy source code    ]   ← Rebuilds on every code change (this is fine and expected)
```

---

## Stage 2 — Runtime (Only What's Needed to Run)

```dockerfile
# ─────────────────────────────────────────────
# STAGE 2 — runtime
# ─────────────────────────────────────────────
# A completely fresh base image — zero build tools carried forward.
FROM python:3.11-slim AS runtime

ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1
ENV PORT=8000
# ↑ Replace 8000 with YOUR_APP_PORT if different

# Install ONLY what the application needs at runtime.
# Each package below has a specific reason — remove what your project doesn't use.
RUN apt-get update && apt-get install -y --no-install-recommends \
    libpq5 \          # PostgreSQL shared library (.so file) needed by psycopg2 at runtime.
                      # Note: this is NOT libpq-dev. That was compile-time headers.
                      # This is the actual runtime shared library. Different thing.
    tesseract-ocr \   # OCR engine binary — remove if you don't process images/PDFs
    poppler-utils \   # PDF CLI tools (pdftotext, pdftoppm) — remove if not needed
    curl \            # Used by healthcheck probes defined in docker-compose.yml
    && rm -rf /var/lib/apt/lists/*

# 🔒 Create a non-root user — NON-NEGOTIABLE FOR PRODUCTION
# If a vulnerability is ever exploited, the attacker gets limited user
# permissions instead of full root access to your host machine.
RUN useradd --create-home --shell /bin/bash appuser
# ↑ Replace "appuser" with any name — just keep it non-root

WORKDIR /app

# This single line is the magic of multi-stage builds.
# The entire compiled venv — all your Python packages — gets copied here.
# The ~200MB of build tools from Stage 1 stay behind. They never arrive.
COPY --from=builder /venv /venv

# Copy your application code.
# Rule of thumb: only copy what actually needs to run.
# Never copy: .env, Dockerfile, docker-compose.yml, .git, __pycache__
COPY main.py          ./        # ← YOUR_MAIN_ENTRYPOINT (main.py, app.py, run.py, etc.)
COPY your_app/        ./your_app/  # ← YOUR_APP_PACKAGE_FOLDER (app/, src/, etc.)
# Add more COPY lines for other folders your project needs (e.g., configs/, assets/)

# Create writable directories for uploads, model cache, and outputs.
# Mount these as named volumes in docker-compose.yml so data persists across restarts.
RUN mkdir -p /app/data /app/outputs /app/.cache \
    && chown -R appuser:appuser /app /venv
    # ↑ Give the non-root user ownership of everything it needs to write

# Switch to the non-root user from this point forward.
USER appuser

ENV PATH="/venv/bin:$PATH"

# ── Three Startup Options — Pick One ────────────────────────────────────
# You can set a default here and override it per-service in docker-compose.yml.

# DEVELOPMENT  → uvicorn with --reload. Watches for file changes, restarts automatically.
#               Use this together with a volume mount so your local code changes apply.
# CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000", "--reload"]

# PRODUCTION   → gunicorn manages multiple worker processes and handles crashes gracefully.
# CMD ["gunicorn", "-c", "gunicorn_conf.py", "main:app"]

# DEFAULT      → single uvicorn worker, no file watching. Good for local testing.
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]

# EXPOSE documents which ports this container uses.
# It does NOT publish them to your host — that's handled by docker-compose.yml.
EXPOSE 8000    # ← YOUR_API_PORT
EXPOSE 8501    # ← YOUR_DASHBOARD_PORT — remove this line if you have no dashboard
```

---

## Placeholders — Replace These With Your Own Values

| Placeholder | What to put here |
|---|---|
| `python:3.11-slim` | Your Python version — `3.10-slim`, `3.12-slim`, etc. |
| `YOUR_APP_PORT` | The port your API listens on. Default is `8000`. |
| `YOUR_DASHBOARD_PORT` | Dashboard port. Streamlit default is `8501`. |
| `YOUR_MAIN_ENTRYPOINT` | Your `main.py`, `app.py`, or `run.py` |
| `YOUR_APP_PACKAGE_FOLDER` | Your `app/`, `src/`, or whatever your package is called |
| `libpq5`, `tesseract-ocr`, `poppler-utils` | Keep only what your project actually uses at runtime |
| `appuser` | Any non-root username you prefer |

---

## The Mental Model — One Picture

```
┌─────────────────────────────────────────────────────┐
│  STAGE 1 — Builder                                  │
│                                                     │
│  base image + gcc + libpq-dev + build tools        │
│       ↓                                             │
│  pip install all packages → compiled /venv          │
└──────────────────────┬──────────────────────────────┘
                       │  COPY --from=builder /venv /venv
                       │  (only the venv crosses over)
                       ↓
┌─────────────────────────────────────────────────────┐
│  STAGE 2 — Runtime                                  │
│                                                     │
│  fresh base image + libpq5 + tesseract + curl       │
│  + /venv (from Stage 1)                             │
│  + your source code                                 │
│  + non-root user                                    │
│                                                     │
│  → This is your final Docker image                  │
└─────────────────────────────────────────────────────┘
```

---

## What Changes Per Project vs What Never Changes

This is the whole point of treating this as a reusable template. The structure, the multi-stage pattern, the caching trick, the non-root user — none of that ever changes. The only things you touch when starting a new project are in the right column.

| Always stays the same — never touch | Changes per project — your only edits |
|---|---|
| Multi-stage build structure | Python version (`3.11`, `3.12`, etc.) |
| Virtual environment at `/venv` | Your app folder name (`app/`, `src/`, etc.) |
| `COPY --from=builder /venv /venv` | Your entrypoint filename (`main.py`, `app.py`) |
| Layer caching trick (requirements first) | Port numbers |
| Non-root user pattern | Runtime system packages (keep only what you use) |
| `ENV PYTHONDONTWRITEBYTECODE` / `PYTHONUNBUFFERED` | Writable directory names (`/app/data`, etc.) |
| `rm -rf /var/lib/apt/lists/*` cleanup | Startup command (uvicorn target, gunicorn config) |

So in practice, picking up this template for a new project means editing maybe 6–8 lines. Everything else is battle-tested boilerplate you copy as-is.

---

## Quick Summary

```
✅ Multi-stage build    → Smaller image, faster deploys, smaller attack surface
✅ Layer caching        → Copy requirements.txt before source code — saves rebuild time
✅ /venv strategy       → One COPY transfers all compiled packages to Stage 2
✅ Non-root user        → Essential security practice for any production container
✅ COPY --from=builder  → Build tools never reach the runtime image
✅ Runtime packages     → Only install what the running app actually needs
```

---
