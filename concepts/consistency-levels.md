# ⚖️ Distributed Consistency & Quorum (R + W > N)

In distributed systems, consistency levels define how and when updates to a data store become visible to other nodes and users. This is the practical application of the **[CAP Theorem](./cap-theorem.md)**.

> **Staff SRE Insight:** "Consistency is not binary; it’s a spectrum. As an architect, you must choose the level that prevents data corruption while minimizing the 'Consistency Tax'—the latency and availability cost added to every request."

---

## 🏗️ 1. The Quorum Mathematics (R + W > N)
A Quorum is the minimum number of votes that a distributed transaction has to obtain in order to be allowed to perform an operation.

* **N:** Number of replicas (Nodes).
* **W:** Write Quorum (Number of nodes that must acknowledge a write).
* **R:** Read Quorum (Number of nodes that must participate in a read).

### The Formula for Strong Consistency:
$$R + W > N$$

If this condition is met, any read quorum will overlap with any write quorum by at least one node, ensuring that the **latest version** of the data is always returned.

#### Common Configurations:
| Configuration | Performance | Reliability | Use Case |
| :--- | :--- | :--- | :--- |
| **W=1, R=1** | Highest (Low Latency) | Low (Weak Consistency) | Caching, Real-time metrics. |
| **W=N, R=1** | Slow Writes, Fast Reads | High Read Reliability | System configurations. |
| **W=Quorum, R=Quorum** | Balanced | High (Strong Consistency) | **Task Schedulers**, Ledgers, Metadata. |

---

## 🚀 2. Consistency Models: The Spectrum

### 🔴 Strong Consistency
The system guarantees that it will return the most recent write.
* **Mechanism:** Synchronous replication to a quorum of nodes.
* **Cost:** High latency (waits for the slowest node in the quorum).
* **Impact:** If a majority of nodes are down, the system is **Unavailable**.

### 🟡 Eventual Consistency
If no new updates are made, eventually all accesses will return the last updated value.
* **Mechanism:** Asynchronous replication.
* **Cost:** Lowest latency.
* **Problem:** "Stale Reads" — a user might see an old version of a task status right after it was updated.

### 🔵 Causal Consistency
Ensures that operations that are "causally" related are seen by all nodes in the same order.
* **Use Case:** Social media comments or message threads.

---

## 🛡️ 3. Fencing Tokens & Versioning
When using Quorum systems, network partitions can still lead to "Zombie Leaders" (nodes that think they are in charge but are partitioned).

* **Fencing Token:** A monotonically increasing number issued by a consensus service (like Zookeeper/Etcd).
* **Action:** Storage nodes reject any write with a token lower than the last processed one.
* **Reference:** See **[Locking Strategies](./locking-strategies.md)** for implementation details.

---

## 📊 SRE Observability Metrics

* **Replication Lag:** The time delta (ms) between the leader's WAL and the follower's last ACK.
* **Quorum Failure Rate:** Frequency of requests failing because `W` or `R` nodes could not be reached.
* **Stale Read Rate:** Percentage of read requests that returned a version older than the latest known write (monitored via trace-id comparison).

---****
