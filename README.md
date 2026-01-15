# 🏗️ System Design Case Studies: Architecture in the AI Era

Welcome to a curated collection of system design solutions. In an era where **AI is rapidly transforming how we write code**, the true essence of software engineering is shifting. 

Coding is becoming more accessible, but **Software Engineering is evolving.** While AI can generate implementation details, it cannot yet independently navigate the complex trade-offs, architectural thinking, and long-term scalability required for high-stakes, real-world systems.

---

## 🚀 The Vision: Master the "Why" Over the "How"

The purpose of this repository is to help engineers look deeper into technical problems from an **architectural perspective**. As AI takes over the "how" (implementation), we as engineers must master the **"why" (system design)**.

To solve real-world technical problems, one must first understand the fundamental building blocks—the "physics" of data and hardware.

### 🧩 1. The Architectural Vocabulary (Foundations)
Before diving into complex designs, we must master the terms and mechanics that define the real world. In the **[Concepts](./concepts/README.md)** section, I share essential engineering trade-offs:
* **Infrastructure:** [Load Balancing](./concepts/load-balancing.md), [API Gateway](./concepts/api-gateway-bff.md), [Performance Benchmarks](./concepts/infrastructure-performance-benchmarks.md).
* **Data & Integrity:** [Sharding](./concepts/sharding.md), [Consensus](./concepts/consensus-algorithms.md), [Capacity Math](./concepts/capacity-estimation-math.md).
* **AI Ready:** [LSM-Trees](./concepts/storage-lsm-trees.md), [Inverted Index](./concepts/inverted-index.md), [Stream Windowing](./concepts/stream-windowing-watermarks.md).

---

## 🛰️ 2. The Case Study Series (20+ Real-World Solutions)

I am releasing 20+ case studies focusing on decision-making and system integration—skills that remain vital as implementation becomes automated.

### 📣 Messaging & Real-time
* **[WhatsApp-like Messenger](./cases/whatsapp.md)** ⏳
* **[Global Push Notification (FCM)](./cases/push-notifications.md)** ⏳
* **[Scalable Notification System](./cases/notifications.md)** ⏳

### 📊 Analytics & Monitoring
* **[Ad Click Aggregator](./cases/ad-click-aggregator.md)** 🚀 **(Live)**
* **[Distributed Logging Pipeline](./cases/logging-pipeline.md)** ⏳
* **[Real-time Analytics (Flink)](./cases/flink-analytics.md)** ⏳
* **[Distributed Monitoring (TSDB)](./cases/tsdb.md)** ⏳

### 🛠️ Core Infrastructure
* **[Distributed Task Scheduler](./cases/task-scheduler.md)** ⏳
* **[Google Load Balancer (Maglev/Anycast)](./cases/google-lb.md)** ⏳
* **[Distributed Key-Value Store](./cases/kv-store.md)** ⏳
* **[Distributed ID Generation](./cases/id-generation.md)** ⏳
* **[Distributed Lock Manager](./cases/distributed-lock.md)** ⏳

### 📦 Storage & Delivery
* **[Blob/Object Storage (GCS/S3)](./cases/blob-storage.md)** ⏳
* **[Image/Video CDN](./cases/cdn.md)** ⏳

### 📱 Social & Discovery
* **[News Feed / Instagram Feed](./cases/news-feed.md)** ⏳
* **[Scalable Search System](./cases/search-system.md)** ⏳
* **[Top K Service (Heavy Hitters)](./cases/top-k.md)** ⏳

### 💰 Transactions & Traffic
* **[Payment Gateway](./cases/payment-gateway.md)** ⏳
* **[Ticketmaster (Booking System)](./cases/ticketmaster.md)** ⏳
* **[Global Rate Limiter](./cases/rate-limiter.md)** ⏳
* **[Software Distribution (Canary Deploy)](./cases/canary-deploy.md)** ⏳

---

## 🛠️ The "Architectural Thinking" Framework
Every analysis in this repo follows a rigorous process:
1. **Constraint Identification:** Defining the real boundaries (QPS, Latency, Throughput).
2. **The Physics of Data:** Calculating hardware limits before picking a tool.
3. **The Trade-off Matrix:** Evaluating Consistency, Availability, and Latency.
4. **AI-Resilience:** Designing systems that can feed and be managed by AI agents.

**"The code may be generated, but the system is designed."**

---
**Maintained by:** [Timucin Tezcan](https://github.com/timucintezcan)
