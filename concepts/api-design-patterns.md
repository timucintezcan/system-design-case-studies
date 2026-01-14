# 🔌 API Design Patterns: REST, GraphQL, and gRPC

Choosing the right communication protocol is a fundamental architectural decision. It's not about which is "better," but which trade-offs (latency, flexibility, type safety) align with your system's goals.

> [!IMPORTANT]
> **Staff Engineer Insight:** "Never pick a protocol just because it's popular. Use **gRPC** for internal microservices to get high performance and type safety. Use **REST** for public APIs to ensure maximum compatibility. Use **GraphQL** for frontend-heavy applications where clients need to minimize round-trips by fetching custom data shapes."

---

## 🏗️ 1. The Big Three Protocols

### A. REST (Representational State Transfer)
The industry standard for public-facing APIs.
* **Pros:** Ubiquitous, easy to cache at the HTTP level, stateless.
* **Cons:** Over-fetching (getting more data than needed) or Under-fetching (needing multiple calls for one view).

### B. gRPC (Google Remote Procedure Call)
Built on HTTP/2 and Protocol Buffers (Protobuf).
* **Pros:** Extremely fast (binary), strictly typed, supports bidirectional streaming.
* **Cons:** Harder for browsers to consume directly, not human-readable without tools.

### C. GraphQL
A query language for your API.
* **Pros:** Client specifies exactly what they need. No over-fetching.
* **Cons:** Complexity on the server side (N+1 query problem), difficult to cache at the CDN level.

---

## 🛠️ 2. API Versioning & Breaking Changes

A key part of API design is managing change without breaking your clients.

* **Path Versioning:** `/api/v1/users` (Most common).
* **Header Versioning:** `Accept: application/vnd.myapi.v1+json`.
* **Backward Compatibility:** Never remove a field or change its type in a minor version. Only add new, optional fields.

---

## 📊 Comparison Summary

| Feature | REST | gRPC | GraphQL |
| :--- | :--- | :--- | :--- |
| **Payload Format** | JSON / XML | Binary (Protobuf) | JSON |
| **Transport** | HTTP 1.1 / 2 | HTTP/2 Only | HTTP |
| **Contract** | Loose (OpenAPI) | Strict (.proto) | Strict (Schema) |
| **Best For** | Public APIs | Internal Microservices | Mobile / Web Frontends |

---

## 🔗 Related Concepts
* [Context Propagation](./context-propagation.md) (Crucial for tracing these calls)
