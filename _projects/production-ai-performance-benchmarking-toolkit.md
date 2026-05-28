---
layout: project
title: "Production AI Performance Benchmarking Toolkit"
date: 2026-05-27
categories: projects
permalink: /projects/production-ai-performance-benchmarking-toolkit/
---

## Overview

A production-grade Python framework for comprehensively benchmarking RAG systems and AI agents. This toolkit moves beyond "it works" to "it can be trusted" by measuring quality, latency, cost, safety, and observability at scale.

Built as a hands-on series with reusable patterns for baseline datasets, quality evaluation, load testing, regression gates, and continuous observability.

## Problem

AI systems fail silently: they sound correct while hallucinating, feel fast in demos while failing at P99 latency, and cost cents in testing while costing dollars in production. Without systematic benchmarking, teams are flying blind.

This toolkit operationalizes benchmarking by:
- Establishing baseline measurements (latency, cost, quality)
- Running regression checks on every change
- A/B testing prompts, retrieval strategies, and configurations
- Detecting drift in production queries and model behavior
- Exposing metrics for alerting and dashboarding

## What's Built

**Core Benchmarking Pipeline**
- Latency profiling with P50/P95/P99 and time-to-first-token (TTFT)
- Token-level cost tracking and cost-aware query routing
- RAGAS quality evaluation (faithfulness, relevancy, precision, recall)
- Retrieval benchmarking across chunking strategies, BM25, dense search, and hybrid search

**API & Observability**
- Production FastAPI server with `/query`, `/health`, `/metrics` endpoints
- Prometheus-compatible metrics exporter (latency, throughput, cost, errors)
- Memory and resource tracking per query
- MLflow integration for experiment tracking and dashboard logging
- Drift detection with Evidently AI patterns

**Testing & Analysis**
- A/B testing framework for prompt variants and model comparisons
- Regression gates (quality, latency, cost thresholds)
- Safety benchmarking and hallucination detection
- Agent trajectory analysis for multi-step reasoning and tool-call efficiency
- Load testing with Locust (concurrent query simulation)

**Data & Utilities**
- Evaluation dataset management with baseline artifacts
- Prompt registry and experiment tracking
- RAG corpus handling and embedding optimization
- Cost benchmarking dashboard API

## Key Features

| Feature | Implementation |
|---|---|
| Quality Scoring | RAGAS (faithfulness, relevancy, precision, recall) |
| Speed Testing | P50/P95/P99 latency, TTFT, throughput under load |
| Cost Analysis | Per-token tracking, model comparison, cost-aware routing |
| Retrieval Testing | Chunking strategies, BM25, dense search, hybrid search, MRR/NDCG |
| A/B Testing | Prompt variants, retrieval strategies, model configs |
| Regression Testing | Automated quality/latency/cost gates with threshold enforcement |
| Agent Eval | Trajectory analysis, tool-call metrics, multi-step reasoning |
| Observability | Prometheus metrics, MLflow logging, drift detection |
| Load Testing | Locust-based concurrent query simulation |
| Production API | FastAPI with async query handling, memory tracking, error reporting |

## Tech Stack

| Area | Technology |
|---|---|
| **LLM & RAG** | OpenAI API, LangChain, LangChain Community |
| **Vector Store** | FAISS (in-memory, CPU-optimized) |
| **Quality Eval** | RAGAS, Datasets |
| **API Server** | FastAPI, Uvicorn, Pydantic |
| **Load Testing** | Locust |
| **Cost Tracking** | Tiktoken (token counting) |
| **Metrics & Monitoring** | Prometheus Client, Evidently AI |
| **Experiment Tracking** | MLflow |
| **Data Analysis** | Pandas, NumPy, SciPy |
| **CI/CD** | GitHub Actions (AI regression pipeline) |

## How It Works

```
1. Load evaluation dataset
   ↓
2. Run baseline (latency, cost, quality snapshots)
   ↓
3. RAGAS evaluation (faithfulness, relevancy scores)
   ↓
4. A/B test variants (prompts, retrieval strategies, models)
   ↓
5. Track costs (token-level, model comparison)
   ↓
6. Run regression checks (gates against quality/latency/cost thresholds)
   ↓
7. Agent benchmarking (trajectory analysis, tool efficiency)
   ↓
8. Expose Prometheus metrics & log to MLflow dashboard
   ↓
9. Detect drift in production queries
```

## Usage

**Full benchmark pipeline:**
```bash
python scripts/run_full_benchmark.py --fast  # quick run
python scripts/run_full_benchmark.py         # full run
```

**Production API server:**
```bash
python api/app.py
# Exposes:
# - POST /query → latency, cost, memory
# - GET /health → status
# - GET /metrics → Prometheus format
```

**Load testing:**
```bash
locust -f api/locustfile.py --headless -u 100 -r 10
```

## What I Delivered

- **Framework design:** Modular benchmarking architecture (baseline → quality → regression → observability)
- **Evaluation patterns:** RAGAS integration, A/B testing infrastructure, cost tracking
- **Production API:** Async-safe FastAPI endpoint with Prometheus metrics and MLflow logging
- **Regression gates:** Automated quality/latency/cost checks for CI/CD pipelines
- **Agent analysis:** Trajectory parsing and multi-step reasoning metrics
- **Observability:** Drift detection, metric exporting, dashboard integration
- **Documentation:** 13-part Medium series covering each component (day 0 through day 12)

## Links

- **GitHub:** [github.com/yachika-yashu/production-ai-performance-benchmarking-toolkit](https://github.com/yachika-yashu/production-ai-performance-benchmarking-toolkit)
- **Medium Series:** 13-part walkthrough (see `/posts_medium` folder in repo)
