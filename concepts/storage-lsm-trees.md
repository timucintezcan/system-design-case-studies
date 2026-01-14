# 💾 LSM-Trees: Optimized for High Write Throughput

In traditional Relational Databases (RDBMS), B-Trees are used, which require random disk writes. In contrast, **LSM-Trees (Log-Structured Merge-Trees)** turn random writes into sequential writes, making them incredibly fast for write-intensive workloads.

> [!IMPORTANT]
> **Staff Engineer Insight:** "Storage is all about the trade-off between Read vs. Write vs. Space. If your system handles millions of sensor heartbeats or social media posts per second, you don't use a B-Tree; you use an LSM-Tree based storage engine to avoid the 'Disk I/O' bottleneck of random writes."

---

## 🏗️ The Architecture: How It Works

An LSM-Tree doesn't update data in place. Instead, it treats the storage as a series of immutable segments.



### 1. Write Ahead Log (WAL)
Before anything else, the write is appended to a log file on disk. If the system crashes, this log is used to restore the memory state. Since it's an "append-only" operation, it's very fast.

### 2. MemTable (In-Memory Buffer)
The data is then written to an in-memory sorted data structure (like a Red-Black Tree or Skip List). Once this buffer reaches a certain size (e.g., 64MB), it is flushed to disk as an **SSTable**.

### 3. SSTables (Sorted String Tables)
These are immutable files on disk where keys are stored in sorted order. Because they are immutable, we never "update" them; we just write new ones.

### 4. Compaction (The Magic)
Since we have multiple SSTables, a background process called **Compaction** merges them. It picks the latest version of a key and discards old or deleted ones (Tombstones), keeping the storage clean and reads efficient.

---

## ⚡ Why Use LSM-Trees?

* **Sequential I/O:** Disks (especially HDDs, but also SSDs) are much faster at sequential writes than random ones.
* **No Locking for Writes:** Since we only write to memory or append to logs, there's no need for complex page locking like in B-Trees.
* **High Compression:** Because SSTables are immutable and sorted, they can be compressed very efficiently.

---

## 📊 Comparison: LSM-Trees vs. B-Trees

| Feature | LSM-Tree (e.g., Cassandra) | B-Tree (e.g., PostgreSQL) |
| :--- | :--- | :--- |
| **Primary Strength** | Write Throughput | Read Throughput |
| **Write Type** | Sequential (Append) | Random (Update in-place) |
| **Read Complexity** | May check multiple files | Typically a single path in tree |
| **Fragmentation** | Handled by Compaction | Handled by Vacuum/Defrag |
