
# 🔒 Locking Strategies & Concurrency Control

In distributed systems, ensuring data integrity when multiple processes attempt to modify the same resource simultaneously is a core challenge. Choosing the right locking strategy directly impacts system throughput and tail latency ($p99$).

---

## 1. Optimistic Locking (OCC)
Optimistic Concurrency Control assumes that multiple transactions can frequently complete without interfering with each other.

* **Mechanism:** Before committing, each transaction verifies that no other transaction has modified the data it read.
* **Implementation:** Usually implemented using a `version` column or a `last_modified_timestamp`.
* **SQL Example:**
```sql
UPDATE tasks 
SET status = 'RUNNING', version = version + 1 
WHERE task_id = 'uuid' AND version = 5;
```
## 2. Pessimistic Locking
Pessimistic Locking assumes conflicts are likely and prevents them by explicitly locking the resource before processing.

Mechanism: A transaction acquires a lock (e.g., Row-level lock) and holds it until the transaction is committed or rolled back.
Implementation: Using database-specific syntax like SELECT ... FOR UPDATE.
Pros: Prevents retries; guarantees consistency for long-running or high-value operations (e.g., balancing a bank account).
Cons: Risk of Deadlocks; significantly reduces throughput as other threads are blocked waiting for the lock to release.

```sql
BEGIN;

-- Find Key and Lock Key
SELECT * FROM tasks 
WHERE task_id = '550e8400-e29b-41d4-a716-446655440000' 
FOR UPDATE;

-- Lock is Done, Go on the business
UPDATE tasks 
SET status = 'RUNNING', 
    started_at = NOW() 
WHERE task_id = '550e8400-e29b-41d4-a716-446655440000';

COMMIT;
```

### 2.1 Deep Dive: Pessimistic Locking Mechanisms

Pessimistic locking is essential when the cost of a collision is too high (e.g., financial double-spending) or when the conflict rate is so high that Optimistic retries would saturate the CPU.

#### A. Row-Level Locking (SELECT FOR UPDATE)
This is the most common implementation in RDBMS (PostgreSQL, MySQL). It tells the database: *"I am reading this row, and I intend to update it. Do not let anyone else touch it until I am done."*

* **Workflow:**
    1. Transaction A starts.
    2. Transaction A executes `SELECT * FROM accounts WHERE id = 1 FOR UPDATE;`.
    3. Transaction B attempts the same command but is **blocked** (enters a wait state).
    4. Transaction A updates the row and commits.
    5. Transaction B is released and can now proceed.



#### B. Deadlocks: The "Pessimistic" Tax
When two or more transactions hold locks that the others need, a **Deadlock** occurs.
* **Example:**
    * Transaction 1 locks Row A, then tries to lock Row B.
    * Transaction 2 locks Row B, then tries to lock Row A.
    * Both wait forever.
* **Staff Solution:** * **Ordered Locking:** Always acquire locks in the same predefined order (e.g., sort IDs alphabetically before locking).
    * **Lock Timeout:** Use `SELECT ... FOR UPDATE WAIT 5` or `NOWAIT` to fail fast instead of hanging the system.

#### C. Performance Impact: The "Congestion" Effect
Unlike Optimistic locking, Pessimistic locking creates **Queues**.
* **p99 Latency:** As the number of concurrent requests for the same row increases, the wait time grows linearly. This can lead to a **Cascading Failure** where all database connection pools are exhausted by waiting threads.

---

### 2.2 Shared (S) vs. Exclusive (X) Locks

Modern databases use different "modes" of locking to allow some level of concurrency even when pessimistic.

| Lock Type | Name | Can others Read? | Can others Write? | Use Case |
| :--- | :--- | :--- | :--- | :--- |
| **Shared (S)** | Read Lock | ✅ Yes | ❌ No | Consistent reports where data must not change during read. |
| **Exclusive (X)** | Write Lock | ❌ No | ❌ No | Critical updates (Balance transfer, Task status change). |

---

### 2.3 When to use Pessimistic over Optimistic?

In our **Distributed Task Scheduler**, we prefer **Optimistic Locking** for the "Picking up a task" phase to keep throughput high. However, we might switch to **Pessimistic Locking** in these scenarios:

1.  **Inventory Management:** When a task involves decrementing a finite resource (e.g., "Only 1 ticket left").
2.  **Long-Running Transitions:** If the state change logic is complex and involves multiple external API calls before the commit.
3.  **Extremely High Contention:** If a specific "Hot Task" is being hammered by 100+ workers simultaneously, the retry-storm of Optimistic locking would be more expensive than a simple queue (Pessimistic).


---

## 3. Distributed Locking (DLM)

Database seviyesindeki kilitlerin yetersiz kaldığı, yani kilidin birden fazla mikroservis veya farklı veritabanı shard'ları arasında senkronize edilmesi gerektiği durumlarda **Distributed Lock Manager (DLM)** kullanılır.

* **Infrastructure:** Genellikle Redis (Redlock), Etcd veya Zookeeper gibi tutarlılık (consistency) odaklı sistemler üzerine kurulur.
* **Mechanism:** Bir process belirli bir anahtar (key) için kilit talep eder ve bu kilit belirli bir **TTL (Time-to-Live)** süresine sahiptir.



### 🛡️ The Fencing Token Pattern
Dağıtık kilitlerin en büyük riski, bir process'in kilidi aldıktan sonra "Stop-the-world" GC duraksaması veya ağ kesintisi yaşaması ve kilit süresinin dolduğunu fark etmemesidir (Client-side pause).

* **The Solution:** Kilit her verildiğinde, yanında monotonik olarak artan bir **Fencing Token** (versiyon numarası) verilir. Veritabanı, sadece mevcut token'dan daha büyük bir token ile gelen yazma isteklerini kabul ederek "zombi" process'lerin veri bozmasını engeller.



---

## 4. Distributed Semaphore & Lease (Kiralama)

Kilitlerin aksine, bir **Lease**, bir kaynağın belirli bir süre için kullanım hakkını verir. Bu, Scheduler sistemlerinde "Worker" yönetimi için endüstri standardıdır.

* **Use Case:** Bir Worker bir görevi seçtiğinde, o görev üzerinde 30 saniyelik bir **Lease** (veya Visibility Timeout) alır.
* **Fault Tolerance:** Eğer Worker çökerse (crash), kilit (lease) otomatik olarak süresi dolunca düşer ve görev tekrar havuzuna döner. Bu sayede görevlerin `RUNNING` durumunda sonsuza dek asılı kalması engellenir.

---

## 5. Strategy Selection Matrix

| Scenario | Recommended Strategy | Primary Reason |
| :--- | :--- | :--- |
| **High Throughput / Low Conflict** | **Optimistic (OCC)** | Best performance, avoids blocking overhead. |
| **High Conflict / Financial Data** | **Pessimistic (PCC)** | Ensures strict consistency at the cost of latency. |
| **Multi-Service Coordination** | **Distributed (DLM)** | Synchronizes access across independent clusters. |
| **Worker Task Assignment** | **Lease / Visibility Timeout** | Native fault tolerance for distributed workers. |

---

## 📊 SRE Observability & Metrics

Kilitleme stratejilerinin sağlığını izlemek için şu metrikler takip edilmelidir:

* **Lock Contention Rate:** Optimistic locking başarısızlık (retry) oranı.
* **Lock Wait Time:** Pessimistic locking sırasında thread'lerin bekleme süresi ($p99$).
* **Deadlock Count:** Veritabanı tarafından yakalanan ve sonlandırılan döngüsel kilit sayısı.
* **Lease Expiration Rate:** Görevlerin worker hataları nedeniyle kaç kez timeout'a düştüğü.
