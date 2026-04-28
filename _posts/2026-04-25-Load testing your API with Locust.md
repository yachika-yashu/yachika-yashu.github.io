---
layout: post
title: "How I Benchmarked My API: From Guesswork to Real Performance Insights Using Locust"
date: 2026-04-25
image: "locust.png"
description: "Load testing your API with Locust"
---
# 🚀 

## 1. 🧠 The Scenario (Real Problem)
I had a small API built with FastAPI.
Everything _seemed_ fine.
- Endpoints were responding
- No visible errors
- It worked perfectly… when I tested it manually
But then a question hit me:
👉 _“What happens if 100 users hit this API at the same time?”_
I had no answer.
- Would it slow down?
- Would it crash?
- Where is the bottleneck?
That's when I realized — I wasn't testing performance at all.

## 2. 🎯 Goal
By the end of this setup, you'll be able to:
- Simulate real users hitting your API
- Measure performance under load
- Understand how your API behaves at scale
And Tool we'll be using for benchmarking  is **Locust**.

## 3. 📁 Project Structure
```
api-benchmarking/
│
├── main.py           # FastAPI app
├── locustfile.py     # Load testing script
├── requirements.txt
```

## 4. ⚙️ Step-by-Step Implementation
## Step 1: 🧪 Create a Simple API
Your FastAPI app stays the same — no changes needed
## Step 2: ⚡ Install Locust
```
pip install locust
```
If this doesn't work, check:
- Your Python environment (venv/conda)
- `pip` version
## Step 3: 🧑‍💻 Create Load Test Script
Create a file named `locustfile.py`:
```
from locust import HttpUser, task, between

class APIUser(HttpUser):
    wait_time = between(1, 3)  # Simulates user think time

    @task(3)
    def get_items(self):
        self.client.get("/items")

    @task(1)
    def create_item(self):
        self.client.post(
            "/items",
            json={"name": "test item", "price": 100}
        )
```
💡 What's happening here:
- `HttpUser` simulates a real user
- `wait_time` makes behavior realistic
- `@task(3)` means higher probability (read-heavy traffic)
- You can model real-world usage patterns easily
## 🔄 Adapt for Any API (The Magic Part)
**What you ONLY need to change:**
```
# 1. UPDATE YOUR ENDPOINTS (most important)
@task(3)
def get_items(self):
    self.client.get("/YOUR_ENDPOINT_HERE")  # ← Change this

@task(1)
def create_item(self):
    self.client.post("/YOUR_ENDPOINT_HERE", json={"YOUR": "PAYLOAD"})
```
**2. UPDATE THE HOST (Step 6)**
```
http://127.0.0.1:YOUR_PORT
```
**That's it!** Here's the **universal template**: # ✅ Add AS MANY ENDPOINTS as you want!
```

class MyAPIUser(HttpUser):
    wait_time = between(1, 3)

    @task(5)  # Most frequent endpoint
    def search_products(self):
        self.client.get("/search?q=phone")
    
    @task(3)  # Product pages  
    def get_product(self):
        self.client.get("/products/123")
    
    @task(2)  # Add to cart
    def add_to_cart(self):
        self.client.post("/cart", json={"product_id": 123})
    
    @task(1)  # Checkout (rare but critical)
    def checkout(self):
        self.client.post("/checkout", json={"cart_id": 456})
    
    @task(1)  # Profile
    def get_profile(self):
        self.client.get("/profile")
```

**The number = frequency:**
- `@task(5)` = 5x more likely than `@task(1)`
- Perfect for modeling **real traffic patterns**
**10 endpoints? 50 endpoints?** Just keep adding `@task()` methods!
## Step 5: ▶️ Run Everything
Start your API:
```
uvicorn main:app --reload
```
Then run Locust:
```
locust
```
Open:
`http://localhost:8089`

## Step 6: 📊 Configure Test
Fill in:
- **Number of Users:** 100
- **Spawn Rate:** 10
- **Host:** http://127.0.0.1:8000
👉 This means:
- 100 users total
- 10 users added per second
## Step 7: 📈 Analyze Results
Once running, you'll see:
- Requests per second (RPS)
- Response time (median, p95, etc.)
- Failures (if any)
💡 If you notice performance issues:
- Check CPU usage
- Inspect database queries
- Look for blocking code (sync calls in async routes)
## 🧪 Best Practices (Learned the Hard Way)
- Warm up your API before testing
- Don't jump to 1000 users instantly → ramp gradually
- Test endpoints independently
- Use realistic payloads
- Monitor system metrics (CPU, memory, DB connections)
## 🔥 Final Thoughts
Before doing this, I _assumed_ my API was fast.
After benchmarking, I _knew_ where it breaks.
That's a huge difference.
If you're building APIs and not testing them under load,  
you're basically flying blind.