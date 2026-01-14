# 🗄️ Sharding & Partitioning

As data grows into the petabyte scale, a single database node becomes a bottleneck for both storage and throughput. **Sharding** and **Partitioning** are techniques used to distribute data across multiple nodes to ensure the system remains performant, scalable, and resilient.

> **Staff SRE Insight:** "Sharding is the ultimate 'divide and conquer' strategy for data. However, it comes with a high price: you lose the ability to perform easy joins and cross-shard transactions. Always shard by a key that reflects your most common query pattern."

---

## 🏗️ 1. Horizontal Partitioning (Sharding)
Sharding involves breaking up a big table into smaller tables (shards) and putting them on different machines.
- **How it works:** You choose a **Sharding Key** (e.g., `user_id` or `merchant_id`). A hash function or a range map determines which shard stores which data.
- **Pros:** Linear scalability for both reads and writes; smaller indexes per node; fault isolation (one shard failing doesn't take down the whole system).
- **Cons:** Complex "Join" operations; difficulty in maintaining global secondary indexes.

## 🚀 2. Vertical Partitioning
This involves splitting a table by columns rather than rows.
- **Example:** In a "User" table, moving large binary data (profile pictures) to one table and basic metadata (username, email) to another.
- **Pros:** Reduces I/O for common queries; allows different storage optimization for different data types (e.g., SSD for metadata, HDD for blobs).

---

## 🛠️ Sharding Strategies

### A. Hash-Based Sharding
- **Logic:** `hash(key) % number_of_shards`.
- **Best for:** Uniform data distribution across the cluster.
- **Challenge:** Changing the number of shards (Resharding) is very difficult as it typically requires moving a large majority of the data.

### B. Range-Based Sharding
- **Logic:** Store data based on ranges of the key (e.g., A-M in Shard 1, N-Z in Shard 2).
- **Best for:** Range queries where data within a specific interval needs to be retrieved together.
- **Challenge:** Can lead to **Hot Shards** if many users or events fall into the same range (e.g., alphabetical clusters or recent timestamps).

---

## 📊 SRE Performance Metrics
- **Shard Skew:** Monitor the data volume and QPS per shard. If one shard is significantly busier than others, it indicates an unevenly distributed shard key.
- **Cross-Shard Query Latency:** Track the overhead of queries that need to "scatter-gather" data from multiple shards, as these increase latency and resource consumption.
- **Rebalancing Toil:** Measure the engineering time and system stress (CPU/Network) during a resharding or data migration event.
