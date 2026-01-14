# ⚖️ Load Balancing: Distributing Traffic and Ensuring Availability

Load balancing is the process of distributing network traffic across multiple servers. This ensures no single server bears too much demand, improving responsiveness and increasing availability.

> [!IMPORTANT]
> **Staff Engineer Insight:** "A Load Balancer is not just a traffic cop; it's the guardian of system reliability. At scale, you must decide between **L4 (Transport Layer)** for raw speed and **L7 (Application Layer)** for intelligent routing based on HTTP headers or cookies. Choosing the wrong one can lead to 'sticky session' nightmares or inefficient resource usage."

---

## 🏗️ 1. L4 vs. L7 Load Balancing



* **L4 (Transport Layer):** Operates at the connection level (IP and TCP/UDP port). It's incredibly fast because it doesn't look at the message content (no SSL decryption here usually).
* **L7 (Application Layer):** Operates at the content level (HTTP/HTTPS). It can route traffic based on URLs (`/api` vs `/static`), cookies, or specific headers.

---

## 🛠️ 2. Balancing Algorithms

How does the LB decide which server gets the next request?

* **Round Robin:** Sequential distribution. Best when all servers have equal capacity.
* **Least Connections:** Sends traffic to the server with the fewest active sessions. Best for long-lived connections (like WebSockets).
* **Consistent Hashing:** Maps requests to servers based on a hash key (e.g., `user_id`). Crucial for maintaining "stickiness" or when using local caches on the application servers.



---

## 🛡️ 3. Health Checks & Failover

A Load Balancer must know which servers are "alive" to avoid sending traffic to a black hole.
* **Active Health Checks:** The LB sends a periodic heartbeat (ping/HTTP GET) to the server.
* **Passive Health Checks:** The LB observes real traffic; if a server starts returning 5xx errors consistently, it's temporarily removed from the pool.

---

## 📊 Summary Table

| Feature | L4 Load Balancing | L7 Load Balancing |
| :--- | :--- | :--- |
| **Visibility** | IP, Port, TCP Sequence | HTTP Headers, Cookies, URL |
| **Speed** | Ultra Fast (Hardware accelerated) | Slower (CPU Intensive / SSL Term) |
| **Smart Routing** | ❌ No | ✅ Yes (Path-based, Header-based) |
| **Security** | Simple Packet Filtering | WAF, Deep Packet Inspection |
