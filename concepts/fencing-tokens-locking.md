# 🛡️ Fencing Tokens & Optimistic Locking

In distributed systems, ensuring that only one node can modify a resource at a time is surprisingly difficult. Network delays, garbage collection (GC) pauses, and clock skews can make traditional locking mechanisms fail.

> [!IMPORTANT]
> **Staff Engineer Insight:** "Never trust a distributed lock alone. A lock tells you who *should* have access, but a Fencing Token proves who *still* has access. In high-concurrency environments, 'Optimistic Locking' is usually your best friend because it avoids the overhead of blocking while maintaining strict integrity."

---

## 🛑 The Problem: The "Zombie" Client

Imagine Client A acquires a lock to write to a file. Then, Client A hits a long "Stop-the-world" GC pause. The lock expires. Client B acquires the same lock and starts writing. Client A wakes up, doesn't know it lost the lock, and completes its write, overwriting Client B's work.



---

## 🤺 1. Fencing Tokens

A **Fencing Token** is a simple, monotonically increasing number (a counter) issued by a lock service (like Zookeeper or Etcd) every time a lock is granted.

* **How it works:**
    1.  Client A gets lock with **Token: 31**.
    2.  Client A pauses. Lock expires.
    3.  Client B gets lock with **Token: 32**.
    4.  Client B writes to storage with Token 32. Storage records "Last seen token: 32".
    5.  Client A wakes up and tries to write with **Token: 31**.
    6.  **The Reject:** Storage sees 31 < 32 and rejects the write.



---

## ⚡ 2. Optimistic Locking (Version Tracking)

Instead of locking a resource before using it (Pessimistic), we let anyone try to update it, but only if they can prove they have the latest version. This is the most common way to handle concurrency in web applications.

* **Mechanism:** Every record has a `version` or `timestamp` column.
* **The SQL Pattern:**
    ```sql
    UPDATE products 
    SET stock = stock - 1, version = version + 1 
    WHERE id = 123 AND version = 5;
    ```
* **Result:** If another process changed the version to 6 while you were calculating, the `WHERE` clause fails (0 rows updated), and your application must retry the operation.

---

## 📊 Comparison: Pessimistic vs. Optimistic vs. Fencing

| Strategy | When to use? | Impact |
| :--- | :--- | :--- |
| **Pessimistic Locking** | High contention (many clients fighting for 1 item). | High latency (others must wait). Risk of deadlocks. |
| **Optimistic Locking** | Low/Medium contention (collisions are rare). | High throughput (no waiting). Requires retry logic in app. |
| **Fencing Tokens** | When using distributed lock managers (Zookeeper/Etcd). | Maximum safety for cross-service coordination. |

---
