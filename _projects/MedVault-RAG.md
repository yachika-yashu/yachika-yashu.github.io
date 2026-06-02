---
layout: project
title: "MedVault-RAG - production-ready AI research platform for healthcare teams"
date: 2026-05-10
categories: projects
permalink: /projects/MedVault-RAG/
---

<div style="position:relative;padding-bottom:56.25%;height:0;overflow:hidden;border-radius:8px;margin-bottom:2rem;">
  <iframe src="https://www.youtube.com/embed/sCsUNH4PkK8"
    style="position:absolute;top:0;left:0;width:100%;height:100%;border:0;"
    allowfullscreen loading="lazy" title="MedVault-RAG Demo"></iframe>
</div>

## Recruiter Snapshot

| Signal | Details |
|---|---|
| Role fit | AI Engineer, GenAI Engineer, RAG Engineer, Data Solutions Architect |
| Problem | Healthcare teams need cited answers from large PDF libraries without manual search. |
| Built | Agentic RAG platform with hybrid search, reranking, citations, PubMed ingestion, streaming UI, tenant isolation, caching, and observability. |
| Production evidence | FastAPI backend, LangGraph agent, Qdrant, PostgreSQL, Redis, Docker Compose, Nginx, LangSmith traces, auth, tests, and deployment configuration. |
| Why it matters | Shows system design, retrieval quality, secure data scoping, and end-to-end AI product delivery. |

## Overview

MedVault is a full-stack, production-ready AI research platform built for healthcare teams. It solves a real problem in clinical and research workflows: medical knowledge is locked inside hundreds of static PDFs — guidelines, research papers, lab reports — and finding the right information at the right time is slow, error-prone, and entirely manual.

MedVault transforms that passive document library into an active, conversational knowledge base. You ask a clinical question in plain English, and the system searches your vault, retrieves the most relevant evidence, and writes a cited answer in seconds — with inline references pointing to the exact page and document it drew from.

The project was built end-to-end: backend API, AI agent, retrieval pipeline, multi-tenant data layer, real-time frontend, authentication, caching, observability, and production deployment — all containerised and running in Docker.

---

## The Problem It Solves

Standard document search (keyword grep, Ctrl+F) fails in research contexts for three reasons:

1. **Semantic gap** — "myocardial infarction" and "heart attack" mean the same thing but keyword search misses the match.
2. **Multimodality** — Medical papers convey a significant portion of their value through tables, figures, and charts that most text extractors silently discard.
3. **Scale** — A research team accumulates hundreds of PDFs over time. No one can hold all of it in their head. Finding contradictions between studies, or the latest dosing protocol, means hours of manual searching.

MedVault addresses all three. It uses semantic vector search to close the language gap, Docling's layout-aware PDF extraction to preserve tables and figures, and a stateful AI agent that can search, evaluate, and reason across an entire document library in a single query.

---

## Architecture

The system is built around a multi-stage agentic RAG (Retrieval-Augmented Generation) pipeline:

### Ingestion Pipeline
When a PDF is uploaded, it goes through three stages:

1. **Extraction (Docling 2.x)** — Unlike simple text extractors, Docling understands document layout using object detection. It converts PDF to structured Markdown, preserving the hierarchy of headings, paragraphs, tables, and figures. Each image is extracted separately and tagged with a stable ID (`<!-- picture-0 -->`) so the agent can later refer to it.

2. **Chunking** — The structured text is split into ~1500-token chunks with overlap to preserve cross-sentence context without overloading the LLM's reasoning window.

3. **Dual-vector indexing (Qdrant)** — Each chunk is embedded twice: once as a dense vector (OpenAI `text-embedding-3-small`, capturing semantic meaning) and once as a sparse vector (BM25, capturing exact keyword matches). Both are stored in Qdrant alongside a `tenant_id` — a hash of the team code — so cross-tenant data leakage is structurally impossible.

### Query Pipeline
Every incoming question goes through seven stages:

1. **Redis exact-match cache** — if the same question was asked before, return the cached answer instantly without touching the LLM.
2. **Guardrail check** — validates the question is clinically relevant before allowing it to consume compute.
3. **Hybrid search** — queries Qdrant for both dense (semantic) and sparse (keyword) matches concurrently, then fuses results using Reciprocal Rank Fusion.
4. **Cross-encoder reranking** — the top candidates from hybrid search are re-evaluated by `ms-marco-MiniLM-L-6-v2`, a dedicated relevance model that is more accurate than cosine similarity alone.
5. **LangGraph agent** — a stateful reasoning loop (not a linear chain) that can decide to reformulate the query, search again with clinical synonyms, or fetch additional context before committing to an answer. This agentic recovery step significantly improves answer quality on ambiguous or multi-concept questions.
6. **Faithfulness check** — after generation, the answer is scored against the retrieved chunks to verify it is grounded in the source material and not hallucinated.
7. **Semantic cache write** — the result is stored in both Redis (exact) and Qdrant's semantic cache (fuzzy), so semantically similar future questions also benefit from the cache.

### Multi-Tenancy
Every user is linked to a `team_code`. Every operation — ingestion, query, summary, comparison — is scoped to that team's `tenant_id` at the database and vector store level. Isolation is enforced by the infrastructure, not by the LLM prompt, which is the only security model that can actually be trusted.

### Real-Time Streaming
Responses stream token-by-token from the FastAPI backend to the Streamlit dashboard via SSE (Server-Sent Events). The frontend simultaneously renders a progress bar (showing which pipeline stage is active) and a typewriter effect as tokens arrive. This eliminates the perceived latency of waiting 10+ seconds for a full response.

---

## Key Features

### Research Chat
Natural language Q&A over your entire document vault. Every answer includes inline citations (`[1]`, `[2]`) with a references section listing the exact source document, author, and year. The agent can also search Arxiv in real time for papers not in the vault.

### Smart Research
Enter a clinical question — no document upload needed. The app automatically queries PubMed Central, identifies the top 3 relevant open-access papers, downloads and indexes them, then answers the question from that freshly ingested evidence. The entire flow takes under 60 seconds.

### Guideline Reference
Surfaces exact recommendation text from clinical guidelines (WHO, NICE, NHS, etc.) along with the Grade of Evidence for each recommendation. Useful for quickly finding the current first-line treatment for a condition without manually searching a 150-page PDF.

### Literature Review
Synthesizes the current state of research on any topic across the entire vault — identifying where papers agree, where they contradict, and what questions remain open. Produces a structured narrative rather than a list of excerpts.

### Document Comparison
Side-by-side AI comparison of any two documents: agreements, contradictions, and clinically significant differences. Designed for comparing a new lab report against a patient baseline, or evaluating two conflicting studies.

### Analytics Dashboard
Per-team observability: total documents indexed, total queries, token consumption, estimated OpenAI cost, average faithfulness score, and a full query history with per-query latency and token breakdown.

---

## Technical Decisions and Trade-offs

**Why LangGraph over a simple chain?**
Linear chains assume every question can be answered in one retrieval pass. Clinical questions often can't — they require follow-up searches, synonym expansion, or fetching related context from a different document. LangGraph's state machine allows the agent to loop and self-correct, which meaningfully improves answer quality at the cost of slightly higher latency on hard questions.

**Why Docling over PyMuPDF or pdfplumber?**
Standard text extractors read PDFs as streams of characters. Docling runs an object detection model on the page layout, understanding that a rectangle with numbers is a table and an image in the margin is a figure. This matters enormously for medical papers where tables often contain the most important data.

**Why hybrid search over pure semantic search?**
Dense vectors are excellent for meaning but poor at exact recall. If a clinician asks about "SGLT2 inhibitors" or a specific drug dosage, BM25 keyword matching finds it reliably even when the semantic model isn't confident. Combining both and fusing with RRF captures the benefits of both approaches.

**Why cross-encoder reranking?**
The initial hybrid search retrieves the top 20 candidates efficiently. The cross-encoder then re-evaluates all 20 pairs (query, chunk) together, which is computationally heavier but far more accurate than cosine similarity for selecting the final 4–6 chunks sent to the LLM. Fewer, higher-quality chunks means less hallucination and more precise citations.

**Why SSE over WebSockets?**
SSE is unidirectional (server → client), which is all that's needed for streaming tokens. It is simpler to implement, works natively over HTTP/1.1, and doesn't require connection state management. WebSockets would add complexity with no benefit here.

---

## Stack

| Layer | Technology | Why |
|---|---|---|
| API | FastAPI + Gunicorn | Async-native, fast, production-proven |
| AI Agent | LangGraph | Stateful loops, not linear chains |
| LLM | GPT-4o-mini | Best cost/quality ratio for RAG generation |
| Embeddings | text-embedding-3-small | Matryoshka support, fast, cheap |
| Reranker | ms-marco-MiniLM-L-6-v2 | Lightweight cross-encoder, strong precision |
| Vector DB | Qdrant | Native hybrid search, tenant filtering, semantic cache |
| Cache | Redis | Sub-millisecond exact-match cache |
| Database | PostgreSQL | User auth, trace logs, analytics |
| Conversation Store | SQLite | Lightweight, persistent chat history |
| PDF Extraction | Docling + Tesseract OCR | Layout-aware, handles scanned PDFs |
| Frontend | Streamlit | Rapid iteration, SSE streaming support |
| Observability | LangSmith | Full trace visibility into agent reasoning |
| Reverse Proxy | Nginx | SSL termination, rate limiting, WebSocket proxy |
| Containers | Docker Compose | Single-command local and production deploy |

---

## What I Built End-to-End

- Designed and implemented the full multi-stage retrieval pipeline from scratch
- Built the LangGraph agent with agentic recovery, tool use, and faithfulness scoring
- Implemented multi-tenant data isolation at both the PostgreSQL and Qdrant layers
- Engineered the SSE streaming architecture for real-time token delivery with progress state
- Built the PubMed auto-ingestion pipeline (Smart Research) that queries, downloads, indexes, and answers in one flow
- Wrote the full test suite covering all eight feature surfaces
- Configured Nginx, Gunicorn, Certbot, and the production Docker Compose stack for deployment
- Integrated LangSmith observability for agent trace inspection

---

## Links

- **GitHub**: [github.com/yachika-yashu/MedVault](https://github.com/yachika-yashu/MedVault)
- **Demo Video**: [Watch on YouTube](https://youtu.be/sCsUNH4PkK8)
