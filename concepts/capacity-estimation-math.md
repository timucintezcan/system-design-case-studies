# 🧮 Capacity Estimation & Performance Math

System design is not just about drawing boxes; it's about understanding the constraints of physics and hardware. This guide covers the essential calculations needed to estimate scale.

> [!IMPORTANT]
> **Staff Engineer Insight:** "Always calculate your network bandwidth and disk I/O before picking a database. If your data ingress is 10 Gbps and your network card is 1 Gbps, your system will fail regardless of your software architecture."

---

## 🏎️ 1. Throughput & Bandwidth (The Bits vs. Bytes)

The most common mistake is confusing **Bits (b)** with **Bytes (B)**.
* **Network speeds** are measured in bits (Gbps, Mbps).
* **Storage and Memory** are measured in Bytes (GB, MB).

### The Conversion:
* **1 Byte = 8 bits**
* To convert **GB/sec** to **Gbps**, multiply by 8.
* *Example:* If your system writes 2 GB of data per second to storage, you need a network capacity of at least `2 * 8 = 16 Gbps`.

---

## 📈 2. QPS, WPS, and RPS

* **QPS (Queries Per Second):** Usually refers to read load.
* **WPS (Writes Per Second):** Refers to write/ingestion load.
* **TPS (Transactions Per Second):** Refers to atomic operations.

### Peak vs. Average:
Systems are never flat. Always design for **Peak Load**, not average.
* **Standard Rule of Thumb:** Peak QPS ≈ 2x to 5x Average QPS.

---

## 📏 3. Practical Estimations (The "Power of 10")

When estimating on the fly, use approximations to move fast:

| Metric | Per Day | Per Second (Approx) |
| :--- | :--- | :--- |
| **Requests** | 1 Million / day | ~12 req/sec |
| **Requests** | 100 Million / day | ~1,200 req/sec |
| **Requests** | 1 Billion / day | ~12,000 req/sec |

---

## 💾 4. Data Latency Numbers (Every Engineer Should Know)

| Action | Latency |
| :--- | :--- |
| **L1 Cache hit** | 0.5 ns |
| **Main Memory (RAM) reference** | 100 ns |
| **Read 1 MB sequentially from Memory** | 250,000 ns (0.25 ms) |
| **Read 1 MB sequentially from SSD** | 1,000,000 ns (1 ms) |
| **Round trip within same Data Center** | 500,000 ns (0.5 ms) |
| **Send packet CA -> Netherlands -> CA** | 150,000,000 ns (150 ms) |

---

## 📊 Summary Table: Units

| Term | Full Name | Unit | Context |
| :--- | :--- | :--- | :--- |
| **Throughput** | Data Volume / Time | GB/s or Gbps | Network / Disk I/O |
| **Latency** | Time for a single task | ms or μs | User Experience |
| **Scalability** | Ability to handle +load | - | Growth |
