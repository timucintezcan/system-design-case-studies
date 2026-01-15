# 🏎️ Infrastructure Performance Benchmarks

Understanding hardware limits is the only way to avoid "Cloud Waste" and architectural bottlenecks. Below are the updated benchmarks for modern server-grade hardware.

---

## 🏗️ 1. Networking: The Modern "Kernel Wall"
Standard Linux networking has improved, but physical and software limits still apply.

| Component | Standard (2026) | Performance Note |
| :--- | :--- | :--- |
| **NIC Bandwidth** | **25 Gbps - 100 Gbps** | Most cloud instances (AWS/GCP) now offer 25G as default. |
| **Throughput (100G)** | **~12.5 GB/s** | Theoretical max for a 100 Gbps network card. |
| **Kernel PPS Limit** | **~2M PPS** | Standard Kernel interrupt handling limit per core. |
| **eBPF/XDP Limit** | **10M - 20M PPS** | High-speed packet processing bypassing the stack. |
| **Intra-Region Latency**| **< 1 ms** | Latency between two instances in the same AWS Region. |

---

## 💾 2. Storage & Memory: The Latency Hierarchy
Data access speed is the #1 factor in database performance.

| Operation | Latency (Approx) | Context |
| :--- | :--- | :--- |
| **L1 Cache Hit** | **1 ns** | Processor core speed. |
| **L2 Cache Hit** | **10 ns** | 10x slower than L1. |
| **L3 Cache Hit** | **40 ns** | Shared across cores. |
| **Main Memory (RAM)** | **100 ns** | The "Memory Wall" starts here. |
| **NVMe SSD (Gen5) Read**| **10 - 50 μs** | 100x faster than old SATA SSDs. |
| **Regional Network Trip**| **1 - 2 ms** | Data sent across Availability Zones. |
| **Global Network Trip** | **150 ms** | Speed of light limit (NY to London). |

---

## 📊 3. Per-Core Capacity Planning (SRE Guide)
Estimate how many requests a **single modern CPU core** (e.g., AWS Graviton4 or Intel Sapphire Rapids) can handle.

| Load Profile | QPS (per Core) | Request Size | Typical Use Case |
| :--- | :--- | :--- | :--- |
| **Ultra Light** | **1,500,000+** | < 64 B | Redis-style counters, Rate limiting. |
| **Light** | **800,000** | 128 - 512 B | Metric collection, simple KV lookups. |
| **Medium** | **250,000** | 1 KB - 4 KB | API Gateways, Protobuf/JSON parsing. |
| **Heavy (TLS)** | **40,000 - 80,000** | 16 KB - 64 KB | HTTPS Proxying, heavy encryption. |
| **Compute Intensive**| **5,000 - 10,000** | Varies | Image processing, complex AI inference. |

---

## 🛡️ 4. Staff Engineer Hard Limits (Safety Margins)
When designing for 99.99% availability, do not plan for 100% capacity:

* **CPU:** Aim for **50-60%** utilization to handle sudden bursts.
* **Network:** Aim for **70%** of total bandwidth to avoid packet drops.
* **Disk:** Keep **20%** free space to allow the OS to perform maintenance (Compaction, GC).
* **RAM:** Monitor **swap usage**; if it starts swapping, your latency will increase by 1000x.

---
*"Hardware is the floor, Software is the ceiling. You can't reach higher than your floor allows."*
