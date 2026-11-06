# Architectural Review and Evaluation Report: Online Bookstore System

**Course:** CIS 285 — Software Engineering Tools  
**Evaluation Scope:** Base Architecture (Modular Layered 3-Tier MVC) vs. Alternative Architecture (Event-Driven Microservices with Saga Orchestration)

---

## 1. Executive Summary
This evaluation reviews the Base Architecture and Alternative Architecture proposed for the Online Bookstore system. Both proposals preserve the core business invariants: strictly non-negative stock availability, atomic multi-line inventory reservations, idempotent checkout submissions, and safe payment reconciliation[cite: 1]. 

The primary divergence lies in their operational boundaries, data storage models, and inter-component coordination patterns[cite: 1].

---

## 2. Dimensional Evaluation

### 2.1 Scalability & Performance
* **Base Architecture (Layered MVC Monolith):**
  * Scales vertically or horizontally as a uniform unit behind a load balancer.
  * Catalog reads and checkout contention compete for the same execution resources and database connections.
  * Incur zero network latency for internal module boundaries (in-memory invocation).
* **Alternative Architecture (Event-Driven Microservices):**
  * Provides independent horizontal scaling: read-heavy Catalog search replicas can scale independently from transaction-heavy Order and Inventory instances.
  * Asynchronous broker buffering eliminates blocking downstream calls during traffic spikes.
  * Incurs inter-service network overhead and serialization latency.

### 2.2 System Complexity & Operational Overhead
* **Base Architecture:**
  * **Low operational overhead**: Single relational database, straightforward deployment pipeline, unified debugging and local execution.
  * Relies on native ACID database transactions for checkout and inventory updates, eliminating dual-write and split-brain risks.
* **Alternative Architecture:**
  * **High operational overhead**: Requires managing a durable message broker, transactional outbox/inbox tables, consumer offsets, dead-letter queues (DLQs), and distributed tracing (`correlationId`/`causationId`).
  * Introduces distributed eventual consistency across service-private datastores.

### 2.3 Modularity & Component Boundaries
* **Base Architecture:**
  * Implements logical modularity with dependency inversion (Ports & Adapters / Clean Architecture).
  * Enforces inward dependencies toward domain policies, preventing MVC presentation layers from touching storage or computing prices.
* **Alternative Architecture:**
  * Enforces physical encapsulation: each service owns its datastore with no shared tables or joins permitted.
  * Enforces strict event and command contracts via immutable schema versions.

### 2.4 Data Consistency, Transactions, and Fault Tolerance
* **Base Architecture:**
  * Immediate consistency: Checkout, stock reservation, and local state transitions succeed or fail within atomic transactions.
  * Process crashes during remote payment calls leave attempts pending, requiring provider reconciliation.
* **Alternative Architecture:**
  * Eventual consistency: The system coordinates workflows using a long-running Checkout Saga.
  * High fault isolation: Outages in notifications or fulfillment do not block storefront browsing or order creation.
  * Requires explicit compensating actions (e.g., releasing reservations upon downstream payment decline).

---

## 3. Trade-off Comparison Matrix

| Evaluation Criterion | Base Architecture (Modular 3-Tier) | Alternative Architecture (Microservices + Broker) |
|---|---|---|
| **Architectural Style** | In-process Modular Monolith | Distributed Event-Driven Microservices |
| **Data Storage Pattern** | Shared ACID relational database | Private database per service |
| **Transaction Model** | Atomic local transactions | Choreographed/Orchestrated Saga + Outbox |
| **Consistency** | Immediate / Strong | Eventual consistency across domains |
| **Network Overhead** | Negligible (in-memory calls) | Significant (HTTP/gRPC/Broker serialization) |
| **Infrastructure Footprint** | Low (single container/app + DB) | High (broker cluster, 6+ databases, gateway) |
| **Local Debugging & Testing** | Fast, deterministic unit/integration tests | Requires testcontainers, broker mocks, or Docker Compose |
| **Domain Invariant Enforcement** | High (enforced in single domain layer) | High (isolated within authoritative services) |

---

## 4. Evaluation Verdict
For a single-store application and standard academic software engineering scope, the **Base Architecture** offers the optimal architectural fit[cite: 1]. It enforces identical domain boundaries, strict authorization, and inventory invariants as the microservices design without the distributed systems tax (network partitions, outbox relays, and saga compensation recovery)[cite: 1].