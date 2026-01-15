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
