# ⚖️ Consistency Models & CAP Theorem

In a distributed system, you cannot have everything. The **CAP Theorem** states that a distributed data store can only provide two out of three guarantees: **Consistency**, **Availability**, and **Partition Tolerance**.

> **Staff SRE Insight:** "In the real world, 'P' (Network Partition) is not an option—it's a fact of life. Therefore, the real architectural choice is always between **CP** (Consistency) and **AP** (Availability) during a failure. Understanding this trade-off is what separates a working system from a reliable one."

---

## 🏗️ 1. The CAP Pillars

### 🔵 Consistency (C)
Every read receives the most recent write or an error. 
- **Use Case:** Financial systems, inventory management.

### 🟢 Availability (A)
Every request receives a (non-error) response, without the guarantee that it contains the most recent write.
- **Use Case:** Social media feeds, product recommendations.

### 🟡 Partition Tolerance (P)
The system continues to operate despite an arbitrary number of messages being dropped or delayed by the network between nodes.



---

## 🚀 2. The Real Choice: CP vs. AP

Since network partitions are inevitable in distributed systems, we must choose how the system behaves when nodes cannot communicate:

### A. CP (Consistency + Partition Tolerance)
If a partition occurs, the system refuses to respond to requests unless it can guarantee data correctness. 
- **Trade-off:** High Latency or Timeouts during failures.
- **Example:** **Paxos/Raft** based systems, **Google Spanner**.

### B. AP (Availability + Partition Tolerance)
If a partition occurs, nodes remain online and accept writes/reads, even if they cannot sync with each other.
- **Trade-off:** Data may be stale or conflicting (Eventual Consistency).
- **Example:** **Cassandra**, **DynamoDB**, **DNS**.

---

## 🛠️ Consistency Models (The Spectrum)

Consistency is not binary; it exists on a spectrum:

1. **Strong Consistency:** After a write, any subsequent read will return that value. (Expensive, slow).
2. **Eventual Consistency:** If no new updates are made to a key, eventually all reads will return the last updated value. (High performance).
3. **Read-Your-Writes Consistency:** A user will always see their own updates immediately, even if other users see stale data for a short time.



---

## ⚖️ Architectural Decision Matrix

| Model | Latency | Data Freshness | Use Case |
| :--- | :--- | :--- | :--- |
| **Strong (CP)** | High | Guaranteed | Payments, Distributed Locks |
| **Eventual (AP)** | Low | Not Guaranteed | Metrics, Logging, Social Media |

---

## 📊 SRE Performance Metrics

- **Consistency Lag (Staleness):** The time difference between a write on the leader and its visibility on the slowest replica.
- **Quorum Latency:** The p99 time it takes to reach a consensus among $N/2 + 1$ nodes.
- **Conflict Rate:** In AP systems, how often two different values are written to the same key during a partition (requiring resolution like LWW - Last Write Wins).

---
