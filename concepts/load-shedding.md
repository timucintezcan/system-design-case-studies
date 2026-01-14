# 🛡️ Load Shedding

Load Shedding is a critical **Reliability Pattern** used to protect a system from becoming unresponsive when it reaches its capacity limits (**Saturation**). Instead of trying to process every request and eventually crashing the entire system, the system strategically drops "non-critical" traffic to ensure that the "core" services remain healthy.

> **Staff SRE Insight:** "Load shedding is the ultimate survival instinct of a distributed system. It is always better to serve a subset of your users perfectly than to serve all of your users poorly (or not at all) due to a cascading failure."

---

## 🏗️ 1. Why Load Shedding? (The Death Spiral)
When a system is overwhelmed, latency increases. As latency increases, clients often retry (The Thundering Herd), which further increases the load. This leads to a "Death Spiral":
1. **Queue Buildup:** Requests wait in memory buffers.
2. **Resource Exhaustion:** CPU and RAM hit 100%.
3. **Cascading Failure:** The node crashes, shifting its load to other nodes, which then also crash.



---

## 🚀 2. How it Works: Admission Control
Unlike Rate Limiting (which focuses on users), Load Shedding focuses on the **system's local health**. It acts at the entry point of a service to decide which requests to "admit" based on current saturation.

### A. Priority-Based Dropping
We categorize incoming requests by their business importance:
- **Critical (P0):** Core functionality (e.g., Checkout, Login).
- **Important (P1):** User-facing secondary features.
- **Low (P2):** Background tasks, analytics, or "test" metrics.

During saturation, the Load Shedder starts dropping P2, then P1, to protect P0.

---

## 🛠️ Implementation Strategies

### LIFO (Last-In, First-Out) with CoDel
When a system is overloaded, the requests at the front of the queue are likely already timed out on the client side. By processing the **newest** requests first (LIFO) and dropping the old ones, we increase the chance of successful responses.

### Fail-Open vs. Fail-Closed
- **Fail-Open:** If the load shedder itself has an issue, allow all traffic.
- **Fail-Closed:** If the system is unstable (e.g., a Database failure), stop all incoming requests to prevent data corruption.



---

## ⚖️ Architectural Trade-offs
- **Goodput vs. Throughput:** Throughput is the total number of requests processed. **Goodput** is the number of *successful and useful* requests. Load shedding sacrifices throughput to maximize goodput.
- **Client Impact:** It returns an immediate `HTTP 503 (Service Unavailable)` or `429 (Too Many Requests)`, allowing the client to back off early rather than waiting for a 30-second timeout.

---

## 📊 SRE Performance Metrics

- **Request Drop Rate:** Number of requests rejected per priority level.
- **Goodput Ratio:** Successful responses vs. total requests during a peak.
- **Queue Latency (Wait Time):** How long a request sits in the admission queue before being processed or dropped.
- **CPU/RAM Saturation Threshold:** The point (e.g., 80% CPU) where load shedding begins to trigger.

---
