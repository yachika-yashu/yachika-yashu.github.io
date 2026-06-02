---
layout: project
title: "ResearchHub - AI Research Assistant"
date: 2026-05-27
categories: projects
permalink: /projects/research-hub/
---

<div style="background:#FFFDF8;border:1px solid #ececec;border-left:4px solid #f58506;border-radius:6px;padding:16px 20px;margin:0 0 1.5rem;">
  <div style="font-size:13px;font-weight:700;text-transform:uppercase;letter-spacing:0.6px;color:#f58506;margin-bottom:6px;">Project Demo</div>
  <p style="margin:0 0 12px;color:#555;line-height:1.6;">Watch a walkthrough of ResearchHub — paper chat, hybrid retrieval, literature reviews, and the research workflows in action.</p>
  <div style="position:relative;width:100%;padding-bottom:56.25%;height:0;border-radius:6px;overflow:hidden;">
    <iframe src="https://www.youtube.com/embed/pc9u-_JEkc8" title="ResearchHub demo" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen style="position:absolute;top:0;left:0;width:100%;height:100%;"></iframe>
  </div>
</div>

## Recruiter Snapshot

| Signal | Details |
|---|---|
| Role fit | AI Engineer, Research AI Engineer, Full-Stack GenAI Engineer |
| Problem | Research teams lose time switching between PDFs, notes, citation managers, search tabs, and manual literature reviews. |
| Built | Self-hosted AI research assistant with paper chat, hybrid retrieval, bulk ingestion, literature reviews, comparison, notes, BibTeX export, Arxiv monitoring, and team isolation. |
| Production evidence | FastAPI, Streamlit, LangGraph, Qdrant, PostgreSQL, Redis, async workers, Docker, CI, tests, load tooling, and cost tracking. |
| Why it matters | Shows product thinking, research workflow design, retrieval engineering, and deployment-ready implementation. |

## Overview

ResearchHub is a production-grade, self-hosted AI research assistant for academics, engineers, and research teams. It helps users upload papers, chat with their document vault, run literature reviews, compare papers, monitor Arxiv, export BibTeX, and track notes and usage in one workspace.

The system has evolved from a simple RAG application into a full research operating layer with asynchronous ingestion, hybrid retrieval, structured extraction, knowledge graph views, team isolation, cost tracking, and deployment-ready Docker infrastructure.

## What It Solves

Research workflows are usually fragmented across PDFs, notes, search tabs, citation managers, and manual literature review documents. ResearchHub brings those workflows into one searchable, conversational platform.

It is designed for questions like:

- What is the main contribution of this paper?
- How do these two papers differ on methodology?
- Which papers in my vault share authors or keywords?
- Can I export my current vault as BibTeX?
- What new Arxiv papers match the topics I care about?

## Core Features

- Multi-turn conversational Q&A over uploaded papers with persistent thread memory
- Hybrid dense + sparse retrieval using Qdrant, BM25, and reciprocal rank fusion
- Bulk PDF ingestion with real-time progress tracking
- Auto-generated five-sentence summaries at ingestion time
- Structured extraction of contributions, datasets, baselines, and limitations
- Literature review generation across selected papers
- Head-to-head paper comparison on a specific research question
- Knowledge graph of paper relationships by shared authors and keywords
- Raw passage search without LLM generation
- Reading queue for papers not yet added to the vault
- Arxiv keyword monitoring with daily alerts
- Per-user Markdown notes with preview
- Usage dashboard for token consumption, cost, and faithfulness metrics
- Team-based multi-tenancy so each team sees only its own vault

## Architecture

```text
Browser
  -> Streamlit dashboard
  -> FastAPI backend
      -> LangGraph research agent
      -> Qdrant hybrid vector search
      -> PostgreSQL users, notes, summaries, checkpoints
      -> Redis cache and Pub/Sub
  -> arq worker
      -> PDF extraction with Docling and OCR fallback
      -> Chunking, embeddings, Qdrant upsert
      -> Summary and structured field extraction
      -> Arxiv daily monitor
```

## Technical Highlights

### Hybrid Retrieval

The retrieval layer combines dense semantic search with sparse BM25 search and fuses results with RRF. This improves recall for both semantic questions and exact technical terms.

### Background Ingestion

PDF parsing, chunking, embedding, metadata extraction, and summarization run through an async worker. The frontend receives progress updates instead of blocking during long ingestion jobs.

### Research-Specific Tools

The app goes beyond chat. It includes BibTeX export, literature review synthesis, paper comparison, structured paper fields, raw passage retrieval, and a knowledge graph view.

### Multi-Tenant Isolation

Users are grouped by team code. Each vector search, note, paper, summary, and usage record is scoped to the relevant team.

### Production Readiness

The repo includes Docker Compose, production Compose, Caddy HTTPS configuration, CI workflow, tests, a Makefile, load testing scripts, and profiling utilities.

## Stack

| Layer | Technology |
|---|---|
| Backend | FastAPI, LangGraph, LangChain |
| LLM | OpenAI GPT-4o-mini |
| Embeddings | OpenAI text-embedding-3-small |
| Vector Search | Qdrant hybrid dense + sparse search |
| Database | PostgreSQL, SQLAlchemy, psycopg3 |
| Cache and Queue | Redis, arq |
| Frontend | Streamlit |
| Document Processing | Docling, pytesseract fallback |
| Deployment | Docker Compose, Caddy, GitHub Actions |
| Testing and Performance | Pytest, Locust, profiling scripts |

## What I Built

- Designed the end-to-end research assistant workflow
- Built the FastAPI backend and Streamlit dashboard
- Implemented async ingestion with worker-based PDF processing
- Added hybrid search and LangGraph tool orchestration
- Built research workflows for literature review, paper comparison, BibTeX export, and Arxiv monitoring
- Implemented team isolation, authentication, usage tracking, and deployment configuration

## Links

- **Demo:** [Watch on YouTube](https://youtu.be/pc9u-_JEkc8)
- **GitHub:** [github.com/yachika-yashu/Research-hub](https://github.com/yachika-yashu/Research-hub)
