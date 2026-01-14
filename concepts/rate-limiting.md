# 🛡️ Rate Limiting: Protecting System Availability

Rate limiting is the practice of limiting the number of requests a user or service can make within a given timeframe. It is the first line of defense against DoS attacks and "noisy neighbor" problems in multi-tenant systems.

> [!IMPORTANT]
> **Staff Engineer Insight:** "Rate limiting isn't just for security; it's for **fairness**. In a distributed system, a single misconfigured script from one user can starve all other users of resources. Implementing rate limiting at the **API Gateway** level is mandatory for system stability."

---

## 🏗️ 1. Common Algorithms

### A. Token Bucket
A "bucket" holds tokens. Each request costs one token. Tokens are added back at a fixed rate.
* **Pros:** Allows for occasional "bursts" of traffic.
* **Cons:** Simple, but needs memory to store bucket state per user.

### B. Leaky Bucket
Requests enter a bucket and leave at a constant rate (like a leaking bucket).
* **Pros:** Smoothes out traffic spikes completely.
* **Cons:** Does not allow for bursts; requests are dropped if the bucket is full.

### C. Fixed Window Counter
Counts requests in a fixed timeframe (e.g., 100 requests per minute).
* **Cons:** The "Edge Problem"—a user can send 100 requests at 00:59 and another 100 at 01:01, effectively doubling the limit in seconds.

---

## 🛠️ 2. Distributed Rate Limiting

In a microservices world, rate limiting must be global.
* **The Solution:** Use a central fast store like **Redis** to keep the counters.
* **The Risk:** Checking Redis for every single request adds latency. 
* **Optimization:** Use "Local Batching"—each instance rate-limits locally and syncs with Redis every few hundred milliseconds.

---

## 📊 Summary Table

| Algorithm | Allows Bursts? | Complexity | Smoothness |
| :--- | :--- | :--- | :--- |
| **Token Bucket** | ✅ Yes | Low | ⚠️ Moderate |
| **Leaky Bucket** | ❌ No | Low | ✅ High |
| **Fixed Window** | ❌ No | Very Low | ❌ Low (Edge problem) |
| **Sliding Window**| ❌ No | High | ✅ High |
