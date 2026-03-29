# Monolith Architecture Template

**Version:** 1.0
**Owner:** Architect Agent (Team 2)
**Last Updated:** 2026-03-29

---

## Purpose

This template provides the standard blueprint for well-structured monolithic applications. A monolith is not a failure mode — it is the correct architecture for many systems, especially those with small teams, simple domains, or early-stage products where requirements are still evolving. This template ensures the monolith is modular, testable, and decomposable when the time comes.

---

## 1. When to Use a Monolith

**Strong signals for monolith:**
- Single team (1-8 engineers) owns the entire system
- Domain boundaries are unclear and still being discovered
- Rapid iteration and feature velocity are the top priority
- Operational maturity is low (limited infrastructure, monitoring, deployment tooling)
- System is an MVP, prototype, or early-stage product
- Strong consistency requirements across the domain

**Signals against monolith:**
- Multiple independent teams need to deploy independently
- Parts of the system have radically different scaling requirements
- Technology diversity is required (different languages, runtimes)
- System has grown beyond a single team's cognitive capacity

---

## 2. Modular Monolith Design

### The Golden Rule
A monolith must be internally modular even though it deploys as a single unit. Modules have clear boundaries, explicit dependencies, and interact through defined interfaces — not by reaching into each other's internals.

### Module Structure

```
src/
  modules/
    orders/
      api/              # HTTP controllers, request/response DTOs
      domain/           # Entities, value objects, domain services
      application/      # Use cases, command/query handlers
      infrastructure/   # Repository implementations, external integrations
      events/           # Domain events published by this module
      index.ts          # Public API — the ONLY entry point other modules use
    payments/
      api/
      domain/
      application/
      infrastructure/
      events/
      index.ts
    inventory/
      ...
    users/
      ...
  shared/
    kernel/             # Shared value objects, base classes (keep minimal)
    infrastructure/     # Cross-cutting: logging, auth middleware, error handling
    config/             # Application configuration
  main.ts              # Application entry point, module wiring
```

### Module Boundary Rules

1. **Public API only**: Modules interact only through their public API (exported from `index.ts`). No importing from internal module paths.
2. **No circular dependencies**: If Module A depends on Module B, Module B must not depend on Module A. Use events for reverse communication.
3. **Own your data**: Each module owns its database tables/collections. Other modules read through the module's API, not through direct SQL.
4. **Shared kernel is minimal**: The `shared/kernel` contains only truly universal concepts (Money, DateRange, UserId). If it grows large, boundaries are wrong.
5. **Module-level testing**: Each module has its own test suite that can run independently.

### Enforcing Module Boundaries

**Static analysis (mandatory):**
- Lint rules that prevent cross-module internal imports
- Dependency graph analysis in CI (fail on circular dependencies)
- Architecture fitness functions that verify boundary compliance

**Tools:**
- TypeScript: `eslint-plugin-import` with path restrictions
- Java: ArchUnit
- .NET: NetArchTest
- Go: Package-level visibility (built into language)
- General: Dependency-Cruiser, Madge

---

## 3. Data Architecture

### Single Database, Module-Owned Schemas

```sql
-- Each module owns a schema
CREATE SCHEMA orders;
CREATE SCHEMA payments;
CREATE SCHEMA inventory;
CREATE SCHEMA users;

-- Tables are prefixed or namespaced
CREATE TABLE orders.orders (...);
CREATE TABLE orders.order_items (...);
CREATE TABLE payments.payments (...);
CREATE TABLE payments.refunds (...);
```

**Rules:**
- Each module owns its schema — no other module writes to it
- Cross-module queries go through the module's application layer
- Foreign keys across module boundaries are avoided (use IDs as references)
- Database migrations are organized per module

### Transaction Boundaries

Within a monolith, you have the luxury of ACID transactions across modules:
- Use this when strong consistency is required
- But design as if you might not have it (eases future decomposition)
- Prefer module-internal transactions + domain events for cross-module coordination

---

## 4. Communication Between Modules

### Synchronous (Direct Method Calls)

The simplest approach: Module A calls Module B's public API directly.

```typescript
// orders/application/create-order.ts
import { InventoryModule } from '../../inventory';

async function createOrder(command: CreateOrderCommand) {
  const available = await InventoryModule.checkAvailability(command.items);
  if (!available) throw new InsufficientInventoryError();
  // ... create order
}
```

### Asynchronous (In-Process Events)

For cross-module notifications where the caller does not need a response:

```typescript
// orders/application/create-order.ts
async function createOrder(command: CreateOrderCommand) {
  const order = Order.create(command);
  await orderRepository.save(order);
  await eventBus.publish(new OrderPlaced({ orderId: order.id, items: order.items }));
}

// inventory/application/event-handlers.ts
eventBus.subscribe(OrderPlaced, async (event) => {
  await inventoryService.reserveItems(event.items);
});
```

**Event Bus Implementation:**
- Start with an in-process event bus (simple pub/sub)
- Events are dispatched after the database transaction commits
- If a consumer fails, the event is logged and retried (or sent to a DLQ)
- This pattern directly maps to a distributed event bus when decomposing later

---

## 5. API Design

### REST API Structure

```
/api/v1/orders          # Order module endpoints
/api/v1/payments        # Payment module endpoints
/api/v1/inventory       # Inventory module endpoints
/api/v1/users           # User module endpoints
```

### API Layer Rules
- Controllers are thin — they validate input, call the application layer, format output
- Request/response DTOs are separate from domain models
- Error responses follow a standard format (RFC 7807 Problem Details)
- API versioning strategy defined upfront (URL path or header)
- OpenAPI specification generated from code or maintained alongside it

---

## 6. Scaling a Monolith

### Vertical Scaling
- Increase CPU, memory, and I/O capacity of the single deployment
- Simplest approach, effective up to substantial loads
- Ceiling: single-machine limits and single-database bottlenecks

### Horizontal Scaling (Stateless Monolith)
- Run multiple instances behind a load balancer
- Requires: stateless application (no in-memory session state)
- Session state in external store (Redis, database)
- File uploads to object storage (S3, GCS), not local disk
- Background jobs via a distributed job queue, not in-process timers

### Read Replicas
- Route read-heavy queries to database read replicas
- Application-level read/write splitting
- Acceptable staleness must be defined per use case

### Caching Strategy
- **Application cache**: In-memory cache (Redis) for hot data
- **Query cache**: Materialized views for complex aggregations
- **HTTP cache**: CDN for static assets, Cache-Control for API responses
- Cache invalidation strategy must be explicit for every cached entity

---

## 7. Background Processing

### Job Queue Architecture
- Use a job queue library (BullMQ, Celery, Hangfire, Sidekiq)
- Jobs are serializable, idempotent, and retryable
- Each module defines its own job types
- Failed jobs are retried with exponential backoff
- Dead jobs are captured and alerted on

### Scheduled Tasks
- Cron-like scheduling through the job queue (not OS cron)
- Leader election if running multiple instances (only one processes scheduled jobs)
- Monitoring: alert if a scheduled job misses its execution window

---

## 8. Observability

### Logging
- Structured JSON logging with standard fields
- Request-scoped correlation IDs propagated through all log entries
- Module name included in every log entry
- Log levels: ERROR (action required), WARN (degraded), INFO (business events), DEBUG (diagnostic)

### Metrics
- Request rate, error rate, and latency per endpoint (RED metrics)
- Database query count and duration per request
- Background job queue depth, processing time, failure rate
- Business metrics per module (orders created, payments processed)

### Health Checks
- `/health/live` — process is running
- `/health/ready` — database connected, migrations applied, dependencies reachable
- `/health/startup` — initialization complete

---

## 9. Testing Strategy

| Level | Scope | Speed | Coverage |
|-------|-------|-------|----------|
| Unit | Module domain logic | <1s | 90%+ |
| Integration | Module + database | <5s | All data access |
| Module | Full module stack | <10s | Critical paths |
| API | HTTP endpoint contracts | <5s | All endpoints |
| E2E | Full system user journeys | <30s | Top 10 journeys |

### Module Isolation in Tests
- Each module's tests use an isolated database schema
- Test fixtures are module-specific
- Cross-module integration tests verify the event-based communication

---

## 10. Preparing for Decomposition

Design the monolith so it can be decomposed into services later if needed:

### Decomposition Readiness Checklist
- [ ] Module boundaries enforced by static analysis
- [ ] No cross-module direct database access
- [ ] Cross-module communication uses events (even if in-process)
- [ ] Each module has its own test suite
- [ ] API routes are namespaced by module
- [ ] Database schemas are separated by module
- [ ] Shared kernel is minimal and well-defined
- [ ] Module dependency graph is acyclic

### Extraction Process
When a module needs to become a service:
1. Extract the module's database schema to a separate database
2. Replace in-process event bus with message broker for that module
3. Replace direct method calls with HTTP/gRPC calls
4. Deploy as an independent service
5. Keep the rest of the monolith unchanged

---

## 11. Anti-Patterns to Avoid

| Anti-Pattern | Description | Consequence |
|-------------|-------------|-------------|
| Big Ball of Mud | No module boundaries, everything depends on everything | Impossible to change, test, or reason about |
| Distributed Monolith Pretending | Microservices that share a database and deploy together | All the costs of distribution with none of the benefits |
| Premature Decomposition | Splitting into services before understanding domain boundaries | Wrong boundaries are 10x more expensive to fix in microservices |
| Shared Mutable State | Global variables, static singletons holding request state | Concurrency bugs, impossible horizontal scaling |
| Database-Driven Integration | Modules reading/writing each other's tables | Invisible coupling, impossible schema evolution |

---

*Start with a well-structured monolith. Decompose only when forced by team scaling, independent deployment needs, or divergent scaling requirements. The modular monolith is not a stepping stone — it is a legitimate long-term architecture.*
