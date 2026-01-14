# 🛡️ API Gateway & Backend-for-Frontend (BFF)

In a microservices architecture, clients (Web, Mobile, IoT) should not talk to dozens of services directly. An **API Gateway** acts as a single entry point, while **BFF** tailors that entry point for specific platforms.

> [!IMPORTANT]
> **Staff Engineer Insight:** "Don't turn your API Gateway into a 'Monolith Gateway' by putting business logic inside it. Its job is cross-cutting concerns: Auth, Rate Limiting, and Routing. If your Mobile team needs a different data shape than your Web team, use the **BFF pattern** to decouple their requirements."

---

## 🏗️ 1. API Gateway Responsibilities
* **Authentication & Authorization:** Verifying who the user is before the request hits internal services.
* **Rate Limiting:** Protecting downstream services from abuse (Reference: [Rate Limiting](./rate-limiting.md)).
* **Protocol Translation:** Converting public REST/HTTP calls to internal gRPC/Protobuf calls.
* **Request Aggregation:** Combining results from multiple services into a single response to reduce mobile radio usage.

## 📱 2. The BFF (Backend-for-Frontend) Pattern
Instead of one "God Gateway," you create small gateways for each client type.
* **BFF Mobile:** Optimized for low bandwidth, returns minimal JSON.
* **BFF Web:** Returns full data for desktop users.

---

## 📊 Summary: Gateway vs. BFF

| Feature | API Gateway | BFF |
| :--- | :--- | :--- |
| **Scope** | Global (Entire System) | Client-Specific (Mobile/Web) |
| **Ownership** | Platform/SRE Team | Frontend/Product Team |
| **Logic** | Security, Routing | Data Transformation, UI Optimization |
