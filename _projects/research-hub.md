---
layout: project
title: "ResearchHub - AI Research Assistant"
date: 2026-05-27
categories: projects
permalink: /projects/research-hub/
---

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

- **GitHub:** [github.com/yachika-yashu/Research-hub](https://github.com/yachika-yashu/Research-hub)
