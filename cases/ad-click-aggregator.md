# Case Study: Ad Click Aggregator

> 🏛️ **Methodology:** This analysis is conducted using the **[Hybrid Staff Architect Framework](../FRAMEWORK.md)**. 
> It balances Product Engineering with SRE/NALSD principles to ensure physical feasibility at scale.

---

## 1. Context, Requirements & API Design

### Problem Statement
In digital advertising, tracking and aggregating user clicks is critical for billing (advertisers pay per click) and real-time campaign optimization. The challenge is to build a system that can ingest millions of raw ad-clicks per second and provide aggregated counts (e.g., clicks per ad per minute) with extremely high accuracy and low latency.

### Functional Requirements
* **Ingestion:** Capture raw click events from global users via mobile and web SDKs.
* **Aggregation:** Provide aggregated click counts per `ad_id` for specific time windows using **[Stream Windowing](../concepts/stream-windowing-watermarks.md)** (1-min, 5-min, 1-hour).
* **Querying:** Enable internal dashboards and billing systems to query aggregated results via an API.
* **Accuracy:** Ensure **[Exactly-Once](../concepts/delivery-guarantees.md#-3-exactly-once-semantics-eos)** processing semantics to prevent financial discrepancies (over-billing or under-billing).

### API Design
The system requires a high-throughput ingestion endpoint and a low-latency query endpoint.

#### 1. Ingestion API (High-scale)
* **Endpoint:** `POST /v1/clicks`
* **Description:** Used by the ad-sdk or edge proxies to report a click.

```json
{
  "ad_id": "string",
  "user_id": "string",
  "timestamp": "long (unix)",
  "ip_address": "string",
  "user_agent": "string"
}
```
#### 2. Query API (Aggregated Data)
* **Endpoint:** `GET /v1/stats?ad_id={id}&window=1m&start={t1}&end={t2}`
* **Description:** Used by dashboards to retrieve time-series click data.

```json
{
  "ad_id": "string",
  "window": "1m",
  "data": [
    {"timestamp": 1736952000, "click_count": 12500},
    {"timestamp": 1736952060, "click_count": 13200}
  ]
}
```

### Scale Assumptions & Capacity Math
Based on **[Capacity Math](../concepts/capacity-estimation-math.md)** and **[Performance Benchmarks](../concepts/infrastructure-performance-benchmarks.md)**, we define our technical boundaries:

* **Peak WPS (Writes):** 5 Million clicks / second.
* **Average WPS:** 2 Million clicks / second.
* **QPS (Reads):** 10,000 requests/second (Mostly internal dashboards and automated billing).
* **Data Payload Size:** ~500 Bytes per raw click event.
* **Throughput Requirement:**
  * $5M \times 500B = 2.5\ GB/s$ payload.
  * In terms of network: $2.5\ GB/s \times 8 \approx 20\ Gbps$ (excluding headers).
* **Daily Raw Volume (Calculation):**
  $$5 \times 10^6 \text{ clicks/sec} \times 86,400 \text{ sec/day} \times 500 \text{ bytes} \approx 216 \text{ TB / day (Raw)}$$
  This high volume necessitates a robust **[Data Tiering](../concepts/storage-tiering.md)** strategy to manage storage costs.


## 2. SLO, NFR & Data Modeling

### SLOs (Service Level Objectives)
* **Availability:** **99.99%** for the Ingestion API. Every click is revenue; downtime directly impacts the bottom line.
* **Latency:** * **Ingestion:** $p99 < 50ms$ to ensure mobile SDKs can "fire-and-forget" without blocking client resources.
    * **Query:** $p99 < 200ms$ for internal dashboard responsiveness.
* **Durability:** **99.999999999% (11 nines)**. Once a click is acknowledged by the ingestion layer, it must never be lost.
* **Consistency:** * **[Eventual Consistency](../concepts/cap-theorem.md#%EF%B8%8F-consistency-models-the-spectrum)** is acceptable for real-time dashboards (a few seconds of lag is expected).
    * **Strong Consistency** is required for the final daily billing settlement and financial reporting.

### Non-Functional Requirements (NFR)
* **Fault Tolerance:** The system must survive a zonal failure without data loss or significant performance degradation.
* **Idempotency:** The ingestion layer must support **[Idempotency](../concepts/idempotency.md)** to handle SDK/Proxy retries without double-counting clicks.
* **Scalability:** Must scale horizontally to handle 10x peak traffic during high-traffic events (e.g., Black Friday or global sports events).

### Data Modeling
To balance massive write throughput with fast analytical queries, we adopt a multi-layered storage strategy.

#### 1. Message Bus (Hot Layer)
* **Technology:** **[Kafka](../concepts/backpressure-flow-control.md)** or similar distributed log.
* **Partition Key:** `ad_id` (Ensures all clicks for the same ad are processed by the same consumer for local state aggregation).
* **Retention:** 7 days for replayability.

#### 2. Serving Layer (OLAP)
* **Technology:** ClickHouse or Druid (Optimized for **[LSM-Trees](../concepts/storage-lsm-trees.md)** and high-speed columnar scans).
* **Schema (Aggregated Table):**

| Column | Type | Description |
| :--- | :--- | :--- |
| `ad_id` | `String` | Partition Key / Primary Key |
| `window_start` | `Timestamp` | Start of the **[Stream Window](../concepts/stream-windowing-watermarks.md)** (1m/5m) |
| `click_count` | `UInt64` | Total number of clicks in the window |
| `unique_users` | `HyperLogLog` | Probabilistic cardinality for unique user estimation |

#### 3. Deep Archive (Cold Layer)
* **Technology:** **[Blob Storage (S3/GCS)](../concepts/storage-tiering.md)**.
* **Format:** Columnar Parquet/Avro for cost-efficient batch reconciliation and historical audits.

### Trade-off: CAP Theorem
We prioritize **Availability** and **Partition Tolerance (AP)** for the ingestion path. Rejecting a click due to a consistency check results in direct revenue loss. We bridge the gap to **Strong Consistency** via background reconciliation and a batch processing layer for final billing.

---
