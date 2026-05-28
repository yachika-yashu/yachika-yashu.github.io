---
layout: project
title: "Online Purchase Orders Risk Classification"
date: 2026-05-27
categories: projects
permalink: /projects/online-purchase-orders-classification/
---

## Overview

Online Purchase Orders Classification is a supervised machine learning project that predicts default-payment risk for online purchase orders. The dataset contains roughly 30,000 orders and 44 attributes covering customer behavior, payment method, order history, and risk indicators.

The business goal is to help an online trader identify high-risk customers before fulfillment, reducing financial loss while preserving a reproducible modeling workflow.

## Problem

Each order is classified as:

- `yes`: high risk of default payment
- `no`: low risk of default payment

The model is optimized as a binary risk-ranking problem, with ROC-AUC used as the primary metric.

## Workflow

```text
Raw risk dataset
  -> Domain-aware cleaning
  -> Feature engineering
  -> Stratified train-test split
  -> ColumnTransformer preprocessing
  -> Model comparison
  -> GridSearchCV tuning
  -> XGBoost final model
  -> Persisted sklearn pipeline
```

## Key Features

- Handles missing values using business context instead of blind imputation
- Converts card expiry, last order date, birthdate, and order time into model-ready features
- Preserves cleaned data in Parquet-style workflow to protect data types
- Uses train-only fitting for preprocessing to avoid data leakage
- Applies ColumnTransformer for numeric and categorical features
- Compares Logistic Regression, Random Forest, Gradient Boosting, and XGBoost
- Uses GridSearchCV with 3-fold cross-validation
- Selects XGBoost based on ROC-AUC
- Persists the full preprocessing + model pipeline with joblib

## Model Results

| Model | Accuracy | Precision | Recall | F1 | ROC-AUC |
|---|---:|---:|---:|---:|---:|
| Logistic Regression | 0.942 | 0.922 | 0.942 | 0.915 | 0.736 |
| Random Forest | 0.942 | 0.887 | 0.942 | 0.914 | 0.745 |
| Gradient Boosting | 0.942 | 0.909 | 0.942 | 0.914 | 0.742 |
| XGBoost | 0.942 | 0.945 | 0.942 | 0.914 | 0.749 |

## Stack

| Area | Technology |
|---|---|
| Data | Pandas, Parquet-style saved dataset |
| ML | Scikit-learn, XGBoost |
| Preprocessing | ColumnTransformer, OneHotEncoder, StandardScaler |
| Evaluation | ROC-AUC, precision, recall, F1 |
| Tuning | GridSearchCV |
| Persistence | Joblib |

## Links

- **GitHub:** [github.com/yachika-yashu/Online-Purchase-Orders-Classification](https://github.com/yachika-yashu/Online-Purchase-Orders-Classification)
