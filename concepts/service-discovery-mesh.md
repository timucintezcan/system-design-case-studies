# 📡 Service Discovery & Service Mesh

In dynamic environments like Kubernetes, instances are created and destroyed constantly. **Service Discovery** helps services find each other, while **Service Mesh** manages how they talk to each other.

> [!IMPORTANT]
> **Staff Engineer Insight:** "Service Discovery is the 'Phonebook' of your system. Service Mesh is the 'Security Guard and Accountant' that stands next to every service. Use a Mesh (like Istio) when you need deep observability, mTLS encryption, and advanced traffic shifting (Canary releases) without changing your application code."

---

## 🏗️ 1. Service Discovery Models
* **Client-Side:** Client queries a Service Registry (e.g., Netflix Eureka) and picks an IP.
* **Server-Side:** Client talks to a Load Balancer, which queries the Registry (e.g., Kubernetes DNS / AWS ALB).

## 🕸️ 2. Service Mesh (Sidecar Pattern)
A Service Mesh deploys a "Sidecar Proxy" (like Envoy) next to every service instance.
* **Control Plane:** Manages policies and configuration.
* **Data Plane:** Handles actual traffic, retries, and encryption.

---

## 📊 Comparison: Why use a Mesh?

| Feature | Basic Service Discovery | Service Mesh (Sidecar) |
| :--- | :--- | :--- |
| **Connectivity** | IP/DNS Lookup | Circuit Breaking, Retries |
| **Security** | Manual Auth | mTLS (Automatic Encryption) |
| **Observability** | Basic Logs | Distributed Tracing & Gold Metrics |
