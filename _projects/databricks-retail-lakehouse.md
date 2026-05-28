---
layout: project
title: "Databricks Retail Lakehouse"
date: 2026-05-27
categories: projects
permalink: /projects/databricks-retail-lakehouse/
---

## Overview

Databricks Retail Lakehouse is an end-to-end data engineering project that implements a medallion architecture for a global retail analytics pipeline. It organizes raw customer, product, and transaction data into Bronze, Silver, and Gold layers using Databricks, Spark, Delta Lake, SQL, and PySpark.

The project is framed around a GlobalRetail scenario with large-scale retail operations, multi-region analytics needs, and Power BI-ready reporting.

## Business Goals

- Reduce data processing time from 72 hours to under 6 hours
- Improve inventory forecasting readiness
- Enable customer segmentation and personalization analysis
- Support real-time financial reporting across regions
- Provide clean Gold-layer aggregates for Power BI dashboards

## Architecture

```text
CSV, JSON, Parquet sources
  -> Bronze Delta tables
      raw append-only ingestion with audit timestamps
  -> Silver Delta tables
      validation, cleaning, deduplication, incremental merges
  -> Gold Delta tables
      daily sales and category sales aggregates
  -> Power BI dashboards and business reporting
```

## Key Features

- Bronze, Silver, and Gold Databricks notebooks
- Customer, product, and transaction pipelines
- Incremental merge strategy using Delta Lake ACID transactions
- Email, age, price, stock, transaction, and null validation rules
- Customer segmentation into high, medium, and low value groups
- Product price and stock status categorization
- Daily sales and category-level Gold aggregates
- Power BI-ready output tables
- Documentation for architecture, workflow, and data dictionary
- Governance patterns for access control, audit trail, and data masking

## Stack

| Layer | Technology |
|---|---|
| Platform | Databricks |
| Processing | Apache Spark, PySpark |
| Storage | Delta Lake |
| Query | SQL |
| Architecture | Medallion Lakehouse |
| Visualization | Power BI |
| Documentation | Architecture docs, ETL workflow, data dictionary |

## What I Built

- Designed the medallion pipeline structure
- Created Databricks notebooks for Bronze, Silver, and Gold processing
- Implemented business rules for customers, products, and transactions
- Added documentation for architecture, data dictionary, and ETL workflow
- Prepared Gold-layer tables for BI consumption and executive reporting

## Links

- **GitHub:** [github.com/yachika-yashu/databricks-retail-lakehouse](https://github.com/yachika-yashu/databricks-retail-lakehouse)
