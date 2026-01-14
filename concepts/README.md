# 🏛️ System Design Foundations: Architectural Strategy Map

This directory serves as a comprehensive library of distributed systems concepts. Instead of a simple list, these topics are organized by their role in building, scaling, and maintaining high-performance systems.

---

## 🗺️ Knowledge Index

### 🧱 Infrastructure & Traffic Management
*Building the entry point: How to direct traffic and reduce latency at the edge.*

| Topic | Focus |
| :--- | :--- |
| **[Load Balancing](./load-balancing.md)** | Distributing traffic across L4/L7 layers and consistent hashing. |
| **[API Design](./api-design-patterns.md)** | REST, gRPC, and GraphQL trade-offs and versioning. |
| **[Rate Limiting](./rate-limiting.md)** | Protecting services via Token Bucket, Leaky Bucket, and Distributed limits. |
| **[Health Checks](./health-checks.md)** | Maintaining high availability, failover, and circuit breaking. |
| **[Caching Strategies](./caching-strategies.md)** | Optimization patterns like Cache-aside, Write-through, and Write-behind. |
| **[Cache Eviction](./cache-eviction.md)** | Managing memory constraints with LRU, LFU, and TTL. |
| **[CDN Basics](./cdn-fundamentals.md)** | Global content delivery and edge computing. |

---

### ⚖️ Distributed Data & Integrity
*Moving beyond a single database: Ensuring correctness and atomicity at scale.*

| Topic | Focus |
| :--- | :--- |
| **[Sharding & Partitioning](./sharding.md)** | Horizontal scaling and the "Hot Partition" problem. |
| **[Replication](./replication.md)** | Leader-Follower, Multi-Leader, and Leaderless architectures. |
| **[Index Optimization](./indexing.md)** | B-Trees, Hash Indexes, and physical data layout. |
| **[Consensus](./consensus-algorithms.md)** | Distributed agreement protocols like Paxos and Raft. |
| **[Idempotency](./idempotency.md)** | Ensuring safe retries and state consistency. |
| **[Outbox Pattern & CDC](./outbox-pattern-cdc.md)** | Solving the "Dual-Write" problem for eventual consistency. |
| **[Delivery Guarantees](./delivery-guarantees.md)** | At-Most-Once, At-Least-Once, and Exactly-Once semantics. |
| **[Fencing Tokens & Locking](./fencing-tokens-locking.md)** | Handling concurrency with distributed mutexes and tokens. |

---

### 💾 Storage Engine & Performance Internals
*Understanding the 'Physics' of data to optimize I/O and hardware costs.*

| Topic | Focus |
| :--- | :--- |
| **[LSM-Trees](./storage-lsm-trees.md)** | Optimizing for high-throughput writes (SSTables and MemTables). |
| **[Inverted Index](./inverted-index.md)** | The engine behind full-text search and keyword lookups. |
| **[Storage Tiering](./storage-tiering.md)** | Cost-efficiency through Hot, Warm, and Cold data architectures. |
| **[Probabilistic Structures](./probabilistic-data-structures.md)** | Memory-efficient membership checks using Bloom Filters & Sketches. |

---

### ⚡ Asynchronous Processing & Streaming
*Handling infinite data flows and decoupling system components.*

| Topic | Focus |
| :--- | :--- |
| **[Fan-out Strategies](./fan-out-strategies.md)** | Scaling social graphs and notification systems (Push/Pull models). |
| **[Backpressure & Flow Control](./backpressure-flow-control.md)** | Protecting downstream services from being overwhelmed. |
| **[Stream Windowing](./stream-windowing-watermarks.md)** | Processing late-arriving data with Event-Time and Watermarks. |

---

### 📈 Observability & Operational Excellence
*Gaining deep visibility to ensure reliability and cost-efficiency.*

| Topic | Focus |
| :--- | :--- |
| **[Context Propagation](./context-propagation.md)** | Distributed tracing across microservice boundaries. |
| **[Metrics Cardinality](./metrics-cardinality.md)** | Optimizing monitoring costs and time-series query performance. |
| **[Sampling Strategies](./sampling-strategies.md)** | Intelligent trace selection via Head and Tail sampling. |

---

## 🛠️ How to use this Library
1. **Theoretic Base:** Start here to understand the "Why" before moving to the "How."
2. **Staff Deep-Dives:** Each document contains insights on trade-offs and real-world failure modes.
3. **Application:** Reference these concepts in the `cases/` directory during system design simulations.

> *"A Senior Engineer knows how to use a tool. A Staff Engineer knows why that tool will eventually fail at 100x scale."*
