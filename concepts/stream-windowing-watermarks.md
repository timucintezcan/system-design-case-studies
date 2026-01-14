# ⏳ Stream Windowing & Watermarks: Handling Time in Motion

In batch processing, we have a fixed set of data. In **Streaming**, data is infinite. To perform any calculation (like "average price in the last 5 minutes"), we must slice this infinite stream into finite chunks called **Windows**.

> [!IMPORTANT]
> **Staff Engineer Insight:** "The biggest challenge in streaming isn't the volume; it's the **out-of-order data**. In the real world, a user's mobile event might arrive 10 minutes late due to poor connection. As a Staff Engineer, you must distinguish between **Event Time** (when it happened) and **Processing Time** (when you saw it), and use **Watermarks** to decide when to give up waiting."

---

## 🪟 1. Windowing Types

How do we slice the time?

### A. Tumbling Windows (Fixed-size, No overlap)
Slices time into fixed, non-overlapping segments. 
* **Example:** 00:00-00:05, 00:05-00:10. 
* **Use Case:** Total sales every hour.

### B. Sliding Windows (Overlapping)
Windows that overlap. 
* **Example:** A 5-minute window that starts every 1 minute.
* **Use Case:** 5-minute moving average of a stock price updated every minute.

### C. Session Windows (Activity-based)
Windows defined by periods of activity followed by a gap of inactivity (timeout).
* **Use Case:** Tracking a user's session on a website.



---

## 🌊 2. Watermarks: Handling Late Data

What happens if an event with a timestamp of `10:01` arrives at the server at `10:15`?

* **The Problem:** We can't wait forever for late events, but we want to be as accurate as possible.
* **The Solution:** A **Watermark** is a "clock" inside the stream that tells the system: *"I am reasonably sure no more events older than time X will arrive."*
* **Logic:** If the Watermark is `10:10`, the system closes the `10:00-10:05` window and emits the result. Any data arriving later than the watermark is either dropped or handled in a "Late Side-Output."



---

## ⏱️ 3. Event Time vs. Processing Time

* **Event Time:** The time the action actually occurred on the user's device. (The only source of truth).
* **Processing Time:** The time the message reached your server. (Easier to implement, but inaccurate due to network delays).

---

## 📊 Comparison Summary

| Window Type | Overlap? | Fixed Size? | Primary Use Case |
| :--- | :--- | :--- | :--- |
| **Tumbling** | No | Yes | Reporting (Hourly/Daily totals) |
| **Sliding** | Yes | Yes | Monitoring (Moving averages/Alerts) |
| **Session** | No | No | User behavior analysis |
