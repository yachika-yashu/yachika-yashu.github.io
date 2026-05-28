---
layout: project
title: "ClinETL Pipeline - Clinical Trial Monitoring ETL"
date: 2026-05-27
categories: projects
permalink: /projects/clinetl-pipeline/
---

## Overview

ClinETL Pipeline is a production-style, real-time data engineering system for monitoring a Phase 3 clinical trial across 500+ patients and 10 hospital sites. It ingests wearable and lab data, validates clinical quality rules, processes events through Kafka and Spark Structured Streaming, stores trusted data in Delta Lake and PostgreSQL, and supports anomaly detection, drift detection, adverse event prediction, alerting, and audit logging.

The project is designed to demonstrate healthcare-grade data pipeline thinking: correctness, lineage, monitoring, governance, and operational reliability.

## Problem

Clinical trial operations need fast visibility into patient vitals, lab results, protocol deviations, and adverse-event risk. A batch-only workflow can miss critical signals, while a pipeline without strong validation can create compliance and patient-safety risk.

ClinETL addresses that by combining streaming ingestion, schema validation, data quality checks, ML scoring, and immutable audit trails.

## Architecture

```text
Synthetic patient and trial data
  -> Kafka topics
  -> Spark Structured Streaming
  -> Schema validation and quality checks
  -> Delta Lake and PostgreSQL
  -> ML anomaly, drift, and adverse-event scoring
  -> FastAPI alerting layer and Streamlit dashboard
  -> Governance audit logs
```

## Key Features

- Simulates clinical trial data for 500+ patients across 10 sites
- Streams wearable and lab records through Kafka
- Processes events with Spark Structured Streaming
- Validates records against clinical schema expectations
- Applies range, missingness, consistency, and protocol-deviation checks
- Sends bad records to a dead letter workflow
- Detects anomalies with Isolation Forest
- Monitors drift with statistical tests and Evidently AI patterns
- Predicts adverse-event probability with XGBoost
- Supports alert severity mapping for critical, high, and medium events
- Maintains immutable audit logs aligned with FDA 21 CFR Part 11 thinking
- Documents HIPAA, CDISC SDTM, and clinical governance considerations

## Stack

| Layer | Technology |
|---|---|
| Streaming | Kafka |
| Processing | Spark Structured Streaming |
| Storage | Delta Lake, PostgreSQL |
| Data Quality | Great Expectations patterns, custom validators |
| ML | Scikit-learn, XGBoost, Evidently AI |
| API and Alerts | FastAPI, Slack webhook pattern |
| Dashboard | Streamlit |
| Orchestration | Airflow pattern |
| Deployment | Docker, Docker Compose |

## What I Built

- Designed the clinical trial monitoring architecture
- Implemented ingestion modules for Kafka producer and consumer flows
- Built schema validation and transformation modules
- Added audit logging for governance and traceability
- Structured the project for Docker-based local execution
- Created tests to validate startup and core project wiring

## Links

- **GitHub:** [github.com/yachika-yashu/ClinETL-Pipeline](https://github.com/yachika-yashu/ClinETL-Pipeline)
