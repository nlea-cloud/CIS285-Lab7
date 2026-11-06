# Architectural Consolidation Instructions

## 1. Architectural Decision
**Selected Approach:** **Modular Layered Architecture (Base Architecture)** enhanced with the domain modeling rigor, outbox patterns, and explicit state enumerations defined in the Alternative Architecture[cite: 1].

### Decision Rationale
1. **Scope Appropriateness:** A single bookstore system does not warrant the infrastructure overhead and failure modes of distributed microservices[cite: 1].
2. **Transactional Integrity:** Stock reservations across multiple books remain atomic and deterministic within a single transactional store, eliminating distributed compensation complexity.
3. **Best-of-Both Synthesis:** Retain the unified, single-deployable modular architecture from the Base design, while adopting the explicit event envelopes, message contracts, and state models specified in the Alternative design for future extensibility[cite: 1].

---

## 2. Step-by-Step Instructions for the AI Coding Agent (Codex / Copilot)[cite: 1]

Run the following instructions to generate the final consolidated documentation set:

### Task 1: Generate `requirements.md`[cite: 1]
Create an updated, unified software requirements document containing:
1. **Functional Requirements (FRs):**
   - Customer Storefront: Catalog browsing, full-text book search, persistent shopping cart management, single-attempt checkout submissions.
   - Staff Workspace: Catalog updates, price adjustments, stock adjustments with required audit trail (actor, delta, reason, timestamp).
   - Checkout & Payments: Multi-item atomic inventory reservation, quote validation, tokenized provider payment processing, idempotency protection.
   - Fulfillment: Order status transitions and shipment tracking.
2. **Non-Functional Requirements (NFRs):**
   - Modularity: Zero cross-tier boundary violations (views contain no SQL/business logic; adapters do not compute store policies).
   - Security: Role-based authorization, identity propagation, zero persistence of raw payment card numbers.
   - Data Invariants: Stock availability = `onHand - reserved`; no-oversell guarantee under concurrent access.

### Task 2: Generate `architecture.md`[cite: 1]
Consolidate the architecture into a definitive, unified design:
1. **High-Level Pattern:** Document the logical 3-tier modular architecture (Presentation MVC, Application & Domain, Data & Integration Adapters).
2. **Cross-Cutting Rules:** Include the formal idempotency rules, state machine specifications (`Pending`, `Approved`, `Declined`, `Unknown`), and provider tokenization boundaries.
3. **System Context & Container Diagram:** Produce a clean Mermaid container/component diagram showing the internal boundaries and external provider integrations.

### Task 3: Generate `design.md`[cite: 1]
Produce the low-level design document containing full UML models:
1. **UML Class Diagram (PlantUML format):**
   - Define concrete domain entities: `Customer`, `Book`, `Cart`, `CartLine`, `Order`, `OrderLine`, `InventoryItem`, `InventoryReservation`, `PaymentAttempt`, `Shipment`.
   - Define application service interfaces and repository/adapter ports: `CatalogQueryPort`, `InventoryReservationPort`, `PaymentProviderPort`, `FulfillmentProviderPort`.
   - Specify accurate UML relationships: Composition (`*--`), Aggregation (`o--`), Association (`-->`), and Interface Realization (`..|>`).
2. **Activity Diagrams (PlantUML format):**
   - **Customer Checkout Workflow:** Swimlanes for `Customer`, `Presentation (MVC)`, `CheckoutUseCase`, and `PaymentAdapter`. Explicitly document the branch for payment timeout/unknown status requiring reconciliation.
   - **Staff Inventory Adjustment Workflow:** Swimlanes for `Bookstore Staff`, `Presentation`, and `InventoryUseCase`. Illustrate validation preventing adjustments below currently active reservation counts.