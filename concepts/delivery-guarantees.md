# 📦 Delivery Guarantees: Reliability vs. Performance

In a distributed network, failure is the norm. Messages can be lost, delayed, or duplicated. A **Delivery Guarantee** is the formal contract between a Producer (sender) and a Consumer (receiver) that defines how many times a message will be processed in the face of these failures.

> [!IMPORTANT]
> **Staff SRE Insight:** "Reliability is expensive. As an engineer, your job is to find the cheapest guarantee that satisfies the business requirement. Don't use *Exactly-Once* for a logging system, and don't use *At-Most-Once* for a bank transfer. Every choice is a trade-off between latency, throughput, and correctness."

---

## 🏗️ 1. At-Most-Once (Best Effort)
The message is sent once and never retried. If the producer sends it and the network drops it, or the consumer crashes before processing, the message is lost.

* **Mechanism:** Fire-and-forget. No ACKs (Acknowledgements).
* **Pros:** Minimal latency, no storage overhead on the producer, highest throughput.
* **Cons:** Data loss is expected.
* **Use Case:** High-volume telemetry, real-time metrics, or log streaming where a 1% loss doesn't impact the overall trend.

---

## 🚀 2. At-Least-Once (The Industry Standard)
This ensures that a message will be delivered to the consumer one or more times. It guarantees **no data loss**, but introduces the risk of **duplicates**.

* **Mechanism:** The producer sends a message and waits for an ACK. If the ACK doesn't arrive (timeout), it sends the message again.
* **The Problem:** If the consumer processes the message but the ACK is lost on the network, the producer retries, causing a duplicate.
* **SRE Requirement:** The consumer **must be idempotent** to handle duplicate messages without side effects.
* **Use Case:** Notification systems, Order processing, standard event-driven architectures.

---

## 💎 3. Exactly-Once Semantics (EOS)
The "Holy Grail" of distributed systems. It guarantees that even if a producer retries or a consumer restarts, the final effect on the system state is as if the message was processed exactly once.

* **Mechanism:** Typically achieved through Distributed Transactions (e.g., Kafka Transactions) or Idempotent Storage Updates.
* **Atomic Commits:** Coordinating the commit of the message offset and the database update (often via Two-Phase Commit or similar consensus).
* **Pros:** Maximum data integrity.
* **Cons:** High performance tax (latency), increased complexity, and harder to scale across multiple regions.
* **Use Case:** Financial ledgers, Ad-click billing (where double-counting leads to financial loss).

---

## 📊 Summary Comparison

| Guarantee | Data Loss? | Duplicates? | Latency | Complexity |
| :--- | :--- | :--- | :--- | :--- |
| **At-Most-Once** | Yes | No | Lowest | Simple |
| **At-Least-Once** | No | Yes | Medium | Moderate |
| **Exactly-Once** | No | No | High | High |
