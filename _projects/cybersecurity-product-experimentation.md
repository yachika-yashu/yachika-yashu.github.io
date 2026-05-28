---
layout: project
title: "Cybersecurity Product Experimentation & Retention Analytics"
date: 2026-05-27
categories: projects
permalink: /projects/cybersecurity-product-experimentation/
---

## Overview

This project simulates an end-to-end product analytics and data science workflow for a cybersecurity subscription product. It uses synthetic but realistically structured telemetry data to analyze user journeys, evaluate experiments, estimate causal impact, model churn risk, and translate findings into stakeholder-ready recommendations.

The goal is to demonstrate product analytics thinking when real proprietary SaaS or cybersecurity data cannot be shared publicly.

## Problem

Subscription products need to understand where users drop off, which onboarding changes improve activation, whether feature adoption actually causes retention lift, and which users are at risk of churn.

This project answers those questions across funnel analytics, experimentation, causal inference, churn modeling, and reporting.

## Workflow

```text
Synthetic users, events, subscriptions, and support tickets
  -> Telemetry quality checks
  -> Funnel and retention analysis
  -> A/B test evaluation
  -> Difference-in-differences causal inference
  -> Churn model training and scoring
  -> Reporting tables and stakeholder summary
```

## Key Features

- Simulates product telemetry for a cybersecurity subscription use case
- Analyzes onboarding funnels and product activation
- Measures retention and subscription conversion
- Evaluates A/B test impact on activation
- Applies causal inference to estimate feature adoption impact
- Builds churn scoring workflow
- Runs data quality checks before analytics
- Produces reporting tables and stakeholder-friendly recommendations
- Includes notebooks for exploration and scripts for pipeline execution

## Stack

| Area | Technology |
|---|---|
| Analytics | Python, SQL, Pandas |
| Modeling | Scikit-learn |
| Causal Inference | Statsmodels, difference-in-differences |
| Reporting | Power BI, stakeholder summary notebook |
| App | Streamlit |
| Data | Synthetic users, events, subscriptions, support tickets |

## What I Built

- Created synthetic product telemetry for a realistic cybersecurity subscription workflow
- Built a full analysis path from raw events to business recommendations
- Added funnel, retention, A/B test, causal inference, and churn notebooks
- Created scripts for pipeline execution and reporting table generation
- Documented the metric framework and business case

## Links

- **GitHub:** [github.com/yachika-yashu/experimentation-causal-inference-retention-cybersecurity-product](https://github.com/yachika-yashu/experimentation-causal-inference-retention-cybersecurity-product)
