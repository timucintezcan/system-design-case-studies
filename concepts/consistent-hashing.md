# 🎡 Consistent Hashing

In a traditional distributed system, mapping data to nodes using a simple hash function like `server = hash(key) % n` (where `n` is the number of servers) works well until the cluster size changes. **Consistent Hashing** is a sophisticated technique that minimizes data movement when nodes are added or removed from a cluster.

> **Staff SRE Insight:** "Standard modulo hashing is an operational nightmare for stability. If you add one server to a 10-node cluster using `mod n`, you end up remapping nearly 90% of your data. Consistent Hashing reduces this remapping to only `1/n` of the data, preventing massive 'cache miss' storms and unnecessary data migration."


                          
                           (0)
                      /     |     \
                   /        |        \
                /           |           \
            Node 4      [Key A]        Node 1
             (O)           * (O)
            /                              \
           /                                \
       [Key C]                            [Key B]
          * *
           \                                /
            \                              /
             (O)           * (O)
            Node 3      [Key D]        Node 2
                \           |           /
                   \        |        /
                      \     |     /

---

## 🏗️ 1. The Hash Ring Concept
Consistent Hashing maps both the **objects** (data) and the **nodes** (servers) onto a logical circular space called a **Hash Ring**.
- **The Ring:** Imagine a range of integers from 0 to $2^{32}-1$. The ends of the range are wrapped around to form a circle.
- **Node Placement:** Servers are hashed based on their identifier (e.g., IP or Hostname) and placed at specific points on this ring.
- **Data Placement:** To find which server holds a key, we hash the key and move clockwise on the ring until we encounter the first server. That server "owns" the data.

## 🚀 2. Virtual Nodes (VNodes)
In a basic hash ring, servers might be distributed unevenly, leading to some nodes carrying more load than others (**Hot Spots**).
- **The Solution:** Instead of mapping a server to a single point, we map it to multiple points (e.g., 100 or 200 virtual locations) on the ring using different hash functions.
- **Benefit:** This ensures a more uniform distribution of data and allows for "Weighted Sharding," where more powerful servers can handle more Virtual Nodes.

---

## 🛠️ Key Advantages

### A. Minimal Resharding
When a node fails or a new node is added, only the keys in the immediate neighborhood of that node need to be moved. The vast majority of the cluster remains untouched, ensuring system stability.

### B. Scalability & Elasticity
It enables truly elastic architectures where nodes can join and leave the cluster dynamically without causing system-wide performance degradation.

---

## ⚖️ Architectural Trade-offs
- **Implementation Complexity:** Harder to implement and debug than simple modulo hashing; requires maintaining a sorted data structure (like a binary search tree) of node locations.
- **Memory Overhead:** Maintaining a large number of Virtual Nodes requires more memory on the client-side or load balancer to keep the "Ring" state up to date.
- **Cascading Failure Risk:** If a node fails, its entire load moves to its immediate clockwise neighbor. Without proper **Load Shedding**, this can cause a "domino effect" of failures.

---

## 📊 SRE Performance Metrics
- **Ring Imbalance Factor:** Measuring the standard deviation of load across all nodes. High deviation indicates that the Virtual Node count is too low for the cluster size.
- **Data Migration Volume:** Monitoring the amount of data moved during node membership changes (e.g., during a deployment or a scale-down event).
- **Lookup Latency:** The time required to find the correct node on the ring, typically $O(\log N)$ where $N$ is the number of virtual nodes.
