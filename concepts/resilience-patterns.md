# 🔄 Resilience Patterns: Retries, Backoff, and Jitter

In distributed systems, failures are inevitable but often transient (e.g., a temporary network glitch or a service restart). **Retries** allow a system to recover automatically. However, naive retries can accidentally create a self-inflicted DDoS attack. To prevent this, we use **Exponential Backoff** and **Jitter**.

> **Staff SRE Insight:** "Retries are a double-edged sword. Without a proper strategy, they turn a small hiccup into a 'Thundering Herd' that can melt down your entire infrastructure. Always assume that your retry might be the one that finally breaks the server."

---

## 🏗️ 1. The Strategy: When to Retry?

Not all errors should be retried. As an architect, you must distinguish between:
- **Retriable Errors:** HTTP 429 (Too Many Requests), 503 (Service Unavailable), 504 (Gateway Timeout), or Request Timeouts.
- **Non-Retriable Errors:** HTTP 400 (Bad Request), 401 (Unauthorized), 404 (Not Found). Retrying these is a waste of resources.

---

## 🚀 2. Exponential Backoff

Instead of retrying immediately (which would overwhelm the failing service), we increase the waiting time between each attempt.
- **Example:**
  - Attempt 1: Wait 100ms
  - Attempt 2: Wait 200ms
  - Attempt 3: Wait 400ms
  - Attempt 4: Wait 800ms

This gives the downstream service time to clear its queue and recover from saturation.



---

## 🎲 3. The Secret Ingredient: Jitter

If 1,000 clients all fail at the same time and use the exact same backoff logic, they will all retry at the exact same moment. This creates "spikes" of traffic that can crash the service again. **Jitter** adds randomness to the backoff time.

- **Without Jitter:** All clients retry at exactly 100ms, 200ms, 400ms...
- **With Jitter:** Clients retry at 92ms, 215ms, 380ms... 

This "smoothes out" the traffic over time, allowing the server to process requests more evenly.



---

## 🛠️ Best Practices

1.  **Set a Max Retry Limit:** Never retry indefinitely. Usually, 3 to 5 attempts are sufficient.
2.  **Use an Idempotency Key:** Ensure that retrying a "Write" operation (like a payment) doesn't result in duplicate records.
3.  **Combine with Circuit Breakers:** If retries are failing consistently, the Circuit Breaker should trip to stop further attempts.

---

## ⚖️ Architectural Trade-offs

- **User Experience vs. System Health:** Long backoff times increase the total request latency for the user, but they protect the system's stability. 
- **Cost:** Every retry consumes CPU, memory, and bandwidth. In high-scale systems (10M+ QPS), even a 10% retry rate can double the infrastructure load.

---

## 📊 SRE Performance Metrics

- **Retry Ratio:** (Total Requests - Unique Requests) / Total Requests. Target: < 5%.
- **Success After Retry:** The percentage of failed requests that eventually succeeded due to a retry.
- **Tail Latency (p999):** Measuring the maximum time a user waits when multiple retries and backoffs occur.

---
