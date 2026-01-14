# 🧊 Storage Tiering: Hot, Warm, and Cold Architecture

In large-scale systems, data volume grows exponentially, but its value often decreases over time. Storing all data on high-performance SSDs is financially unsustainable. **Storage Tiering** is the strategy of moving data between different types of storage based on its age, access frequency, and performance requirements.

> [!IMPORTANT]
> **Staff Engineer Insight:** "Engineering is not just about performance; it's about **cost-efficiency**. A Staff Engineer must know when to move data from expensive NVMe drives to cheap S3 buckets. If 90% of your queries target data from the last 7 days, keeping 3 years of data in your primary database is a 'design smell' that leads to operational bankruptcy."

---

## 🏗️ The Tiering Model



### 1. 🔥 Hot Tier (Recent & Frequent)
This tier handles the most recent data that is being actively written and queried.
* **Storage Type:** NVMe SSDs, High-memory instances.
* **Performance:** Ultra-low latency, high IOPS (Input/Output Operations Per Second).
* **Data Age:** Typically last 24-72 hours.
* **Cost:** Very High.

### 2. 🌤️ Warm Tier (Less Frequent)
Data that is no longer being updated but is still queried for reports or recent history.
* **Storage Type:** Standard SSDs or high-density HDDs.
* **Performance:** Moderate latency. Data is still searchable but slower.
* **Data Age:** Last 30-90 days.
* **Cost:** Medium.

### ❄️ 3. Cold Tier (Archive & Rare)
Historical data kept for regulatory compliance or long-term trend analysis.
* **Storage Type:** Object Storage (Amazon S3, Google Cloud Storage), Cold Archive (AWS Glacier).
* **Performance:** High latency (sometimes minutes/hours to retrieve).
* **Data Age:** 90+ days to years.
* **Cost:** Very Low.

---

## ⚡ How it Works: The Data Lifecycle

1.  **Ingestion:** Data lands in the **Hot Tier** for immediate processing and real-time indexing.
2.  **Rollover:** Once an index reaches a certain size or age, it moves to the **Warm Tier**.
3.  **Shrink/Force-Merge:** During the move, data is often compressed or segments are merged to save space.
4.  **Archiving:** Eventually, data is moved to the **Cold Tier** as a read-only snapshot or highly compressed Parquet files.
5.  **Deletion:** After a retention period (e.g., 5 years), the data is automatically purged.

---

## 📊 Comparison Table

| Feature | Hot Tier | Warm Tier | Cold Tier |
| :--- | :--- | :--- | :--- |
| **Media** | SSD / RAM | HDD / Standard SSD | Object Storage (S3/GCS) |
| **Query Speed** | Milliseconds | Seconds | Minutes to Hours |
| **Access Frequency** | Constant | Frequent | Rare / On-demand |
| **Best For** | Real-time Search | Weekly Analytics | Compliance / Audits |
