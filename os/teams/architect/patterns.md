# Architecture Patterns Library

**Version:** 1.0
**Owner:** Architect Agent (Team 2)
**Last Updated:** 2026-03-29

---

## Purpose

This library is the Architect Agent's primary reference for selecting, combining, and applying architectural patterns. Every pattern includes its context, forces, solution shape, trade-offs, and failure modes. Patterns are not rules — they are tools. Applying the wrong pattern to the wrong problem is worse than applying no pattern at all.

---

## 1. Structural Patterns

### 1.1 Hexagonal Architecture (Ports and Adapters)

**Context:** Applications that must remain testable and decoupled from infrastructure concerns (databases, message brokers, HTTP frameworks).

**Forces:**
- Business logic should be testable without infrastructure
- Infrastructure components change more frequently than domain logic
- Multiple interfaces (REST, gRPC, CLI) serve the same domain

**Solution:**
- Domain logic sits at the center with no outward dependencies
- Ports define interfaces the domain requires (inbound) and uses (outbound)
- Adapters implement ports for specific technologies
- Dependency inversion: domain defines the port; infrastructure implements it

**Structure:**
```
domain/           # Pure business logic, zero imports of infrastructure
  models/         # Entities, value objects, aggregates
  services/       # Domain services, use cases
  ports/          # Interface definitions (inbound + outbound)
adapters/
  inbound/        # HTTP controllers, gRPC handlers, CLI
  outbound/       # Database repos, message publishers, external APIs
config/           # Wiring: connects adapters to ports
```

**Trade-offs:**
- (+) Domain is fully testable with in-memory adapters
- (+) Infrastructure changes are isolated to adapter layer
- (-) More files and indirection for simple CRUD operations
- (-) Team must enforce boundary discipline or it degrades over time

**When NOT to use:** Simple CRUD services with no meaningful domain logic.

---

### 1.2 Clean Architecture

**Context:** Systems requiring strict separation between business rules and delivery mechanisms, where the domain is complex enough to justify the layering overhead.

**Forces:**
- Business rules must survive framework changes
- Multiple use cases operate on the same entities
- Testability of business logic is non-negotiable

**Solution:**
- Concentric layers: Entities > Use Cases > Interface Adapters > Frameworks
- Dependency Rule: source code dependencies point inward only
- Each layer has its own data transfer objects — no leaking internal models

**Layers:**
| Layer | Contents | Dependencies |
|-------|----------|-------------|
| Entities | Domain objects, business rules | None |
| Use Cases | Application-specific business rules | Entities |
| Interface Adapters | Controllers, presenters, gateways | Use Cases |
| Frameworks | DB, web, UI, external services | Interface Adapters |

**Trade-offs:**
- (+) Maximum isolation of business logic
- (+) Use cases are explicit and testable
- (-) Significant mapping overhead between layers
- (-) Over-engineered for thin domain logic

---

### 1.3 Vertical Slice Architecture

**Context:** Teams that want to organize code by feature rather than by technical layer, reducing cross-cutting changes.

**Forces:**
- Feature changes touch every horizontal layer (controller, service, repo)
- Different features have different complexity levels
- Team wants to minimize merge conflicts

**Solution:**
- Each feature is a self-contained vertical slice: handler, logic, data access, tests
- No shared service layer — each slice owns its full stack
- Cross-cutting concerns (auth, logging) handled via middleware/decorators
- Slices communicate through well-defined contracts, not shared state

**Structure:**
```
features/
  create-order/
    handler.ts       # HTTP/event handler
    logic.ts         # Business logic
    repository.ts    # Data access
    types.ts         # Request/response types
    tests/           # All tests for this feature
  get-order/
    handler.ts
    logic.ts
    ...
```

**Trade-offs:**
- (+) Feature changes are localized to one directory
- (+) Simple features stay simple; complex features can use richer patterns
- (-) Shared logic requires explicit extraction and dependency management
- (-) Inconsistency risk: each slice may evolve its own internal patterns

---

## 2. Distributed System Patterns

### 2.1 Microservices

**Context:** Large systems with multiple teams needing independent deployment and technology autonomy.

**Forces:**
- Teams must deploy independently
- Different parts of the system have different scaling requirements
- Technology diversity is desired or required
- Conway's Law: system structure should mirror team structure

**Key Design Rules:**
1. Each service owns its data store — no shared databases
2. Services communicate through APIs or events, never direct DB access
3. Each service is independently deployable and testable
4. Service boundaries align with bounded contexts (DDD)
5. Inter-service calls require timeouts, retries, and circuit breakers

**Anti-patterns to avoid:**
- Distributed monolith: microservices that must deploy together
- Shared database: coupling through data instead of contracts
- Synchronous chains: A calls B calls C calls D (latency + failure cascade)
- Nano-services: services so small they add only overhead

See `templates/microservices.md` for full blueprint.

---

### 2.2 CQRS (Command Query Responsibility Segregation)

**Context:** Systems where read and write workloads have fundamentally different characteristics, scaling needs, or data models.

**Forces:**
- Read model and write model have different optimization requirements
- Write operations require strong consistency; reads can tolerate staleness
- Read queries are complex (joins, aggregations) while writes are simple
- Read and write loads scale differently

**Solution:**
- Separate the write model (commands) from the read model (queries)
- Commands go through domain logic with validation and invariant enforcement
- Read models are optimized projections — denormalized, pre-computed, cached
- Synchronization between write and read side via events (async)

**Trade-offs:**
- (+) Read and write sides scale independently
- (+) Read models optimized for specific query patterns
- (+) Write model stays clean and focused on invariants
- (-) Eventual consistency between write and read sides
- (-) Operational complexity: two data stores, synchronization pipeline
- (-) Debugging: must trace across command, event, and projection

**When to apply:** Systems with 10:1 or greater read-to-write ratio, complex query patterns, or different scaling requirements for reads vs. writes.

---

### 2.3 Event Sourcing

**Context:** Systems where the full history of state changes is valuable (audit trails, temporal queries, debugging) and where reconstructing state from events is feasible.

**Forces:**
- Audit requirements demand knowing exactly what changed and when
- Business needs temporal queries ("what was the state on date X?")
- Domain events are the natural language of the business
- Traditional CRUD overwrites state, losing history

**Solution:**
- Store events as the source of truth, not current state
- Current state is derived by replaying events
- Events are immutable and append-only
- Snapshots reduce replay cost for entities with long event histories
- Projections build read-optimized views from event streams

**Implementation Requirements:**
1. Event store with append-only semantics and optimistic concurrency
2. Event schema versioning strategy (upcasting or lazy migration)
3. Snapshot strategy for aggregates with > 100 events
4. Projection rebuild capability (replay from event 0)
5. Dead letter handling for failed projections

**Trade-offs:**
- (+) Complete audit trail for free
- (+) Temporal queries are natural
- (+) Events are excellent integration points
- (-) Event schema evolution is hard and must be planned from day one
- (-) Eventual consistency is inherent
- (-) Steep learning curve for teams used to CRUD

---

### 2.4 Saga Pattern

**Context:** Distributed transactions that span multiple services where traditional two-phase commit is impractical.

**Forces:**
- A business operation requires coordinated changes across services
- Services own their own databases (no distributed transactions)
- Partial failure must be handled gracefully

**Variants:**

**Choreography (Event-driven):**
- Each service listens for events and reacts independently
- Compensating actions undo previous steps on failure
- Best for simple flows with 2-4 services
- Risk: hard to understand and debug as complexity grows

**Orchestration (Coordinator-driven):**
- Central orchestrator directs the saga step by step
- Orchestrator decides next step based on previous results
- Easier to understand, modify, and debug
- Risk: orchestrator becomes a single point of failure and coupling

**Design Requirements:**
1. Every step must have a compensating action
2. Idempotency on all operations (retries are inevitable)
3. Saga state must be persisted (survives restarts)
4. Timeout handling for every step
5. Observability: trace the entire saga end-to-end

---

### 2.5 Transactional Outbox

**Context:** Services that need to update their database AND publish an event atomically, without distributed transactions.

**Solution:**
1. Write the domain change and the outbox event in the same local transaction
2. A separate relay process reads the outbox table and publishes events
3. Relay uses at-least-once delivery; consumers must be idempotent
4. Outbox entries are marked as published after successful send

**Implementation:**
```sql
CREATE TABLE outbox (
  id UUID PRIMARY KEY,
  aggregate_type VARCHAR NOT NULL,
  aggregate_id VARCHAR NOT NULL,
  event_type VARCHAR NOT NULL,
  payload JSONB NOT NULL,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  published_at TIMESTAMPTZ NULL
);
```

---

## 3. Data Patterns

### 3.1 Database per Service
Each microservice owns a private data store. No other service accesses it directly. Data sharing happens through APIs or events. Enables independent schema evolution and technology selection per service.

### 3.2 Polyglot Persistence
Use the best data store for each service's access patterns: relational for transactional data, document for flexible schemas, graph for relationship-heavy queries, time-series for metrics, key-value for caching. Accept the operational cost of multiple database technologies.

### 3.3 API Composition
Build read models by querying multiple services and composing results. Use when CQRS is overkill. Risks: latency accumulation, partial failure handling, data consistency windows.

### 3.4 Change Data Capture (CDC)
Capture row-level changes from database transaction logs and publish as events. Tools: Debezium, AWS DMS. Enables event-driven integration without modifying application code. Useful for legacy system integration.

---

## 4. Reliability Patterns

### 4.1 Circuit Breaker

**States:** Closed (normal) -> Open (failing, fast-fail) -> Half-Open (testing recovery)

**Configuration:**
- Failure threshold: number of failures before opening (default: 5)
- Reset timeout: time before trying half-open (default: 30s)
- Success threshold: successes needed to close again (default: 3)
- Failure definition: which errors count (5xx, timeouts, connection refused)

**Rules:**
- Every external call (HTTP, database, message broker) must have a circuit breaker
- Circuit breaker state must be observable (metrics, health endpoint)
- Fallback behavior must be defined for every circuit breaker

### 4.2 Bulkhead

Isolate components so one failing component cannot consume all resources and cascade to others. Implementations: thread pool isolation, connection pool isolation, process isolation, or service mesh sidecar isolation.

### 4.3 Retry with Exponential Backoff

**Formula:** `delay = base_delay * 2^attempt + random_jitter`

**Rules:**
- Maximum retry count: 3-5 (never infinite)
- Jitter is mandatory (prevents thundering herd)
- Only retry on transient failures (5xx, timeout, connection reset)
- Never retry on 4xx (client errors are not transient)
- Log every retry with attempt number and delay

### 4.4 Rate Limiting

**Algorithms:** Token bucket (burst-friendly), sliding window (smooth), fixed window (simple).

Apply at: API gateway (global), service level (per-service), and client level (per-tenant). Return 429 with Retry-After header. Distinguish between rate limiting and throttling.

### 4.5 Timeout Cascade Prevention

Every service in a call chain must have a shorter timeout than its caller. If Service A has a 5s timeout calling Service B, Service B must have a <5s timeout calling Service C. Without this rule, upstream timeouts fire while downstream is still working, wasting resources.

---

## 5. Observability Patterns

### 5.1 Structured Logging
All logs are JSON. Every log entry includes: timestamp, level, service name, trace ID, span ID, message, and contextual fields. No unstructured string concatenation. Log levels are meaningful: ERROR (needs human attention), WARN (degraded but functional), INFO (significant business events), DEBUG (diagnostic detail).

### 5.2 Distributed Tracing
Propagate trace context (W3C Trace Context) across all service boundaries. Every inbound request starts or continues a trace. Every outbound call (HTTP, gRPC, message publish) propagates the trace. Instrument: HTTP clients, database drivers, message producers/consumers.

### 5.3 RED Metrics (Rate, Errors, Duration)
Every service exposes: request rate (requests/second), error rate (errors/second), and duration distribution (histogram). These three metrics detect most service-level problems. Complement with USE metrics (Utilization, Saturation, Errors) for infrastructure.

### 5.4 Health Check Pattern
Every service exposes `/health` (liveness) and `/ready` (readiness). Liveness: "is the process alive?" Readiness: "can this instance serve traffic?" Readiness checks verify downstream dependencies. Failed readiness removes the instance from the load balancer without killing it.

### 5.5 SLO-Based Alerting
Alert on symptoms (SLO burn rate), not causes. Define error budgets. Alert when the burn rate exceeds 1x over 1 hour (page) or 3x over 5 minutes (page). Avoid cause-based alerts that fire for non-customer-impacting issues.

---

## Pattern Selection Guide

| System Characteristic | Recommended Patterns |
|----------------------|---------------------|
| Complex domain logic | Hexagonal + DDD + Event Sourcing |
| High read/write ratio | CQRS + Read replicas |
| Multi-team ownership | Microservices + Database per Service |
| Audit requirements | Event Sourcing + CDC |
| Simple CRUD | Vertical Slice + Monolith |
| Real-time processing | Event-Driven + CQRS |
| Legacy integration | CDC + Anti-Corruption Layer |
| High reliability | Circuit Breaker + Bulkhead + Retry |

---

*Patterns are tools, not laws. The right pattern depends on the specific forces at play in your system. When in doubt, start simple and evolve toward complexity only when the forces demand it.*
