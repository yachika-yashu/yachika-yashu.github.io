---
layout: post
title: "How I monitor my API using Grafana + Prometheus (with Docker)"
date: 2026-04-25
image: "grafana-cover.png"
description: "Setting up a simple & reproducible monitoring system to track API request volume and response times."
---
## 🧠 The situation
I had a simple API running with FastAPI. It worked, but I had no visibility into its behavior.
- I couldn’t track incoming requests
- I had no latency insights
- I couldn’t detect performance issues
This is fine for small projects—but not for production-level systems.
So I decided to implement a basic monitoring stack using:
- Prometheus (metrics collection)
- Grafana (visualization)
- Docker (reproducibility)
## 🎯 Objective
Set up a simple, reproducible monitoring system to:
- Track API request volume
- Measure response times
- Visualize performance metrics
## 📁 Project structure
project-folder/  
│  
├── app/  
│   └── main.py  
│  
├── Dockerfile  
├── requirements.txt  
│  
├── prometheus/  
│   └── prometheus.yml  
│  
└── docker-compose.yml
## ⚙️ Step 1: Instrument FastAPI
### 1.1 Add metrics support in `main.py`:
```
from prometheus_fastapi_instrumentator import Instrumentator  
Instrumentator().instrument(app).expose(app)
```
### 1.2 Update `requirements.txt`:
```
prometheus-fastapi-instrumentator
```
## 🐳 Step 2: Containerize the application
### Dockerfile (if not already containerized)
```
FROM python:3.11-slim  
WORKDIR /app  
COPY app/main.py .  
COPY requirements.txt .  
RUN pip install --no-cache-dir -r requirements.txt  
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```
## 🔗 Step 3: Define services for prometheus and Grafana with Docker Compose
docker-compose.yml
```
version: "3.8"  
services:  
  fastapi:  
    build:  
      context: .  
      dockerfile: app/Dockerfile  
    container_name: fastapi  
    ports:  
      - "8000:8000"  
    depends_on:  
      - prometheus  
    networks:  
      - monitor-net  
  prometheus:  
    image: prom/prometheus  
    container_name: prometheus  
    volumes:  
      - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml  
    ports:  
      - "9090:9090"  
    networks:  
      - monitor-net  
  grafana:  
    image: grafana/grafana  
    container_name: grafana  
    ports:  
      - "3000:3000"  
    networks:  
      - monitor-net  
networks:  
  monitor-net:  
    driver: bridge
```
## 📊 Step 4: Configure Prometheus
`prometheus.yml`:

```
global:  
  scrape_interval: 15s  
scrape_configs:  
  - job_name: 'fastapi'  
    metrics_path: /metrics  
    static_configs:  
      - targets: ['fastapi:8000']
```

## ▶️ Step 5: Run the system

```
docker-compose up --build
```

## 🌐 Step 6: Verify services
- FastAPI → [http://localhost:8000/](http://localhost:8000/)
- Metrics → [http://localhost:8000/metrics](http://localhost:8000/metrics)
- Prometheus → [http://localhost:9090/](http://localhost:9090/)
- Grafana → [http://localhost:3000](http://localhost:3000)
## 🔍 Step 7: Validate metrics
In Prometheus [http://localhost:9090/](http://localhost:9090/), run:
```
http_requests_total
```
If no data appears, check the `/metrics` endpoint first.
## 📈 Step 8: Connect Grafana to Prometheus
- In Grafana [http://localhost:3000](http://localhost:3000) login 
- Go to **Add Data Source**
- Select **Prometheus**
- Set URL to:
http://prometheus:9090
- Click **Save & Test**
## 📊 Step 9: Build dashboards
This is the fun part.
go to building a dashboard , then Use metrics like:
- `http_requests_total` → to visualize total traffic
- `http_request_duration_seconds_bucket` → to visualize latency distribution
- `http_request_duration_seconds_sum` → to visualize total processing time
From these, you can build panels for:
- request trends
- response time
- performance spikes
Even a simple dashboard already gives way more visibility than before.
## ✅ Outcome
After implementing this:
- API traffic is visible in real time
- Latency can be tracked and analyzed
- System behavior is no longer a black box
## ⚠️ Common issues
- No metrics → check `/metrics`
- Grafana connection fails → verify `prometheus:9090`
- Port conflicts → update `docker-compose.yml`
## 💡 Key takeaway
Monitoring is not optional for production systems.
Even a simple setup using Prometheus and Grafana can significantly improve system visibility and reliability.
## 🔗 Next step
Extend this setup with **alerting** to detect failures automatically.
