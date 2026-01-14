# 📈 Scalability: Vertical vs. Horizontal

Scalability is the ability of a system to handle increased load by adding resources. In large-scale system design, understanding when to scale "up" and when to scale "out" is the first step in architectural decision-making.

> **Staff SRE Insight:** "Scaling is not just about adding machines; it's about managing the complexity that comes with those machines. Horizontal scaling is the only way to reach 'Infinite Scale', but it introduces the challenge of distributed state."

---

## 🏗️ 1. Vertical Scaling (Scaling Up)
Adding more power (CPU, RAM, SSD) to an existing server.
- **Pros:** Simpler to implement; no changes to application logic; low network latency (everything is on one bus).
- **Cons:** Hard hardware limits (there is only so much RAM you can put in one box); Single Point of Failure (SPOF); extremely expensive at the high end.

## 🚀 2. Horizontal Scaling (Scaling Out)
Adding more machines to the resource pool.
- **Pros:** Theoretically infinite scale; high availability (if one node fails, others take over); cost-effective using "commodity hardware."
- **Cons:** Requires a Load Balancer; introduces network latency; requires complex data sharding and consistency strategies.

---

## ⚖️ Architectural Trade-offs
- **Throughput vs. Complexity:** Horizontal scaling increases throughput but significantly raises the complexity of monitoring and deployment.
- **State Management:** Vertical scaling handles state easily. Horizontal scaling requires external "Source of Truth" (like Redis or a Global DB) to manage shared state.

## 📡 Real-World Application (From our Cases)
- **Metrics Ingestion:** Our math showed that 100M WPS is impossible for a single node. We used **Horizontal Scaling** to distribute the CPU-bound compression tasks across 20+ nodes.
- **Load Balancer (Maglev):** Instead of one giant router, we used a cluster of 1250 nodes to handle 100 Tbps of traffic.
- **Object Storage:** We scale horizontally to manage 100 TB/sec throughput across thousands of disk-bound nodes.

---

## 📊 SRE Performance Metrics
- **Resource Saturation:** Monitor CPU/RAM usage. If a single node hits 80% consistently, it's time to scale.
- **Fan-out Latency:** In horizontally scaled systems, monitor the time added by the "Load Balancer -> Node" hop.
