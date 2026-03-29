# Domain Bridge: From Domain-Agnostic to Domain-Specific

> The kernel speaks in system primitives. The OS speaks in software concepts. The domain bridge is the translator.

---

## Purpose

The kernel operates on 11 universal system primitives that describe any system — software, mechanical, organizational, biological. To build software, these abstract primitives must be translated into concrete software concepts that OS teams can act on.

The domain bridge defines this translation. It maps each kernel primitive to its software-domain equivalents and specifies how OS teams interpret and implement each primitive.

This bridge is what makes the kernel reusable. A different domain bridge could map the same primitives to physical engineering concepts, organizational design concepts, or any other domain. The kernel does not change — only the bridge does.

---

## System Primitives to Software Concepts

### 1. Purpose → Product Requirements and User Stories

**Kernel primitive**: The reason the system exists. What it achieves.

**Software translation**:
- Product requirements document (PRD)
- User stories with acceptance criteria
- Jobs-to-be-done framework
- Business objectives and key results (OKRs)

**OS team responsible**: Team 9 (Product)

**Translation rules**:
- Every stated purpose must map to at least one user story.
- Purposes that cannot be expressed as user stories are too abstract — push back to Phase 1.
- Business objectives must be measurable (tied to the success model).

### 2. Boundary → Service Boundaries, API Contracts, Module Boundaries

**Kernel primitive**: What is inside the system vs. outside. Where the system ends.

**Software translation**:
- Service boundary definitions (what each service owns)
- API contracts (the system's external interface)
- Module boundaries within services
- Network boundaries (VPC, subnet, firewall rules)
- Data ownership boundaries (which service owns which data)
- Trust boundaries (authenticated vs. unauthenticated zones)

**OS team responsible**: Team 2 (Architecture), Team 7 (Security)

**Translation rules**:
- Every boundary must have an explicit contract (API spec, schema, protocol).
- Data ownership must be unambiguous — no shared databases across service boundaries.
- Trust boundaries must align with security threat model.

### 3. Inputs → API Requests, Events, User Actions, Data Feeds

**Kernel primitive**: Everything that enters the system from outside.

**Software translation**:
- HTTP/gRPC API requests from clients
- Webhook callbacks from external services
- User interface interactions (clicks, form submissions, gestures)
- Message queue messages from other systems
- Scheduled triggers (cron, time-based events)
- File uploads and data feed imports
- Sensor data or IoT device signals

**OS team responsible**: Team 3 (Backend), Team 4 (Frontend), Team 5 (Data)

**Translation rules**:
- Every input must have a defined schema (request body, event payload, file format).
- Every input must have validation rules (type checking, range checking, sanitization).
- Every input must have authentication and authorization requirements defined.
- Inputs must be enumerated exhaustively — undocumented inputs are a security risk.

### 4. Transformations → Business Logic, Data Processing, ML Inference

**Kernel primitive**: How the system changes inputs into outputs.

**Software translation**:
- Business logic (domain rules, calculations, workflows)
- Data transformations (ETL/ELT, aggregation, enrichment)
- ML model inference (prediction, classification, recommendation)
- State machine transitions
- Validation and enrichment pipelines
- Rendering and presentation logic

**OS team responsible**: Team 3 (Backend), Team 5 (Data), Team 4 (Frontend)

**Translation rules**:
- Each transformation must be traceable to a purpose/requirement.
- Transformations must be idempotent where possible (safe to retry).
- Side effects must be explicit and documented.
- Transformation ordering and dependencies must be defined.

### 5. Outputs → API Responses, Events Emitted, UI Rendered, Reports

**Kernel primitive**: Everything the system produces for the outside world.

**Software translation**:
- HTTP/gRPC API responses
- Events published to message queues or event buses
- Rendered UI (HTML, native views, components)
- Generated reports and exports
- Notifications (email, SMS, push)
- Logs and metrics (observability outputs)
- Webhook calls to external systems

**OS team responsible**: Team 3 (Backend), Team 4 (Frontend), Team 5 (Data)

**Translation rules**:
- Every output must have a defined schema.
- Every output must be traceable to an input or transformation.
- Output formats must be versioned for backward compatibility.
- Error outputs must be as well-defined as success outputs.

### 6. Interfaces → REST APIs, GraphQL, gRPC, WebSockets, Message Queues

**Kernel primitive**: The connection points between the system and external entities, and between internal components.

**Software translation**:
- External APIs: REST, GraphQL, gRPC, WebSocket endpoints
- Internal APIs: service-to-service communication protocols
- Message interfaces: Kafka topics, RabbitMQ queues, SQS queues
- Database interfaces: connection pools, query interfaces, ORM mappings
- File interfaces: S3 buckets, file system paths, FTP endpoints
- UI interfaces: component props, state management contracts

**OS team responsible**: Team 2 (Architecture), Team 3 (Backend)

**Translation rules**:
- Every interface must have a machine-readable contract (OpenAPI spec, protobuf definition, AsyncAPI spec).
- Interfaces must be versioned from day one.
- Internal interfaces are as important as external ones — they are contracts between teams.

### 7. Resources → Databases, Caches, Cloud Services, Compute, Storage

**Kernel primitive**: What the system consumes to operate.

**Software translation**:
- Databases: PostgreSQL, MySQL, MongoDB, DynamoDB, etc.
- Caches: Redis, Memcached, CDN edge caches
- Compute: servers, containers, serverless functions, GPU instances
- Storage: object storage (S3), block storage, file systems
- Cloud services: managed queues, managed ML, managed search
- Third-party APIs: payment processors, email services, auth providers
- Network: load balancers, DNS, CDN, VPN

**OS team responsible**: Team 6 (DevOps), Team 5 (Data)

**Translation rules**:
- Every resource must have a justification (why this resource and not a simpler alternative).
- Resources must be defined as infrastructure-as-code.
- Resource dependencies must be explicit (what happens if this resource is unavailable).
- Cost implications of each resource must be estimated.

### 8. Constraints → Latency, Throughput, Cost, Compliance, Team Size

**Kernel primitive**: Limitations the system must operate within.

**Software translation**:
- Performance constraints: p50/p95/p99 latency, requests per second, concurrent users
- Cost constraints: monthly cloud spend, cost per transaction, infrastructure budget
- Compliance constraints: GDPR, HIPAA, SOC2, PCI-DSS requirements
- Team constraints: team size, skill set, timezone distribution
- Timeline constraints: delivery dates, milestone deadlines
- Operational constraints: deployment frequency, maintenance windows, SLA requirements

**OS team responsible**: Team 9 (Product), Team 7 (Security), Team 6 (DevOps)

**Translation rules**:
- Constraints must be quantified, not vague ("fast" is not a constraint; "p99 < 200ms" is).
- Constraints must be tested (performance tests, compliance audits).
- Conflicting constraints must be surfaced and resolved in the success model.

### 9. Feedback → Monitoring, Alerting, User Analytics, A/B Tests

**Kernel primitive**: How the system observes its own behavior and adjusts.

**Software translation**:
- Monitoring: Prometheus metrics, Datadog, CloudWatch
- Alerting: PagerDuty, OpsGenie, alert rules and escalation policies
- Logging: structured logs, log aggregation (ELK, Loki)
- Tracing: distributed tracing (Jaeger, Zipkin, OpenTelemetry)
- User analytics: product analytics (Mixpanel, Amplitude), session recording
- A/B testing: feature flags, experiment frameworks
- Health checks: liveness probes, readiness probes, synthetic monitoring

**OS team responsible**: Team 6 (DevOps), Team 9 (Product)

**Translation rules**:
- Observability is not optional. Every service must emit metrics, logs, and traces.
- Alerting must be actionable — alerts without runbooks are noise.
- User feedback loops must close (analytics must inform product decisions).

### 10. Failure Modes → Service Outages, Data Corruption, Cascade Failures, Security Breaches

**Kernel primitive**: How the system can break.

**Software translation**:
- Service outages: individual service failure, dependency failure, infrastructure failure
- Data corruption: write conflicts, replication lag, schema migration errors
- Cascade failures: retry storms, connection pool exhaustion, queue backpressure
- Security breaches: unauthorized access, data exfiltration, injection attacks, credential compromise
- Performance degradation: memory leaks, connection leaks, thread starvation
- Configuration errors: wrong environment variables, stale secrets, mismatched feature flags

**OS team responsible**: Team 7 (Security), Team 8 (QA), Team 6 (DevOps)

**Translation rules**:
- Every failure mode must have a containment strategy (circuit breaker, bulkhead, timeout).
- Every failure mode must have a recovery path (retry, failover, manual intervention).
- Cascading failure paths must be identified and broken with isolation.
- Failure modes must be testable (chaos engineering, fault injection).

### 11. Evolution Paths → Migration Strategies, Versioning, Feature Flags, Strangler Fig

**Kernel primitive**: How the system changes over time.

**Software translation**:
- API versioning strategy (URL versioning, header versioning, content negotiation)
- Database migration strategy (zero-downtime migrations, backward-compatible schemas)
- Feature flags for gradual rollout and rollback
- Strangler fig pattern for incremental replacement of legacy components
- Blue-green or canary deployment for safe production changes
- Module extraction strategy (monolith to services migration path)
- Dependency upgrade strategy (automated security patches, major version upgrades)

**OS team responsible**: Team 2 (Architecture), Team 6 (DevOps)

**Translation rules**:
- The system must be designed to change from day one. "We'll refactor later" is not a strategy.
- Every interface must have a versioning strategy before it launches.
- Database schemas must support additive changes without downtime.
- The evolution path must be realistic given the team's constraints.

---

## How Domain Packs Work

The OS's teams collectively ARE the software domain pack. The mapping above is the software domain bridge. It translates kernel primitives into concepts that Team 2 (Architecture), Team 3 (Backend), Team 4 (Frontend), Team 5 (Data), Team 6 (DevOps), Team 7 (Security), Team 8 (QA), and Team 9 (Product) understand natively.

**Other domain packs could exist**:

| Domain | Purpose Becomes | Boundary Becomes | Failure Mode Becomes |
|---|---|---|---|
| Software | Product requirements | Service boundaries | Service outages |
| Physical Engineering | Design specifications | Physical enclosure | Structural failure |
| Organizational Design | Mission statement | Department boundaries | Communication breakdown |
| Curriculum Design | Learning objectives | Course boundaries | Assessment failure |

The kernel does not need to change for any of these. Only the domain bridge changes.

---

## Extensibility: Creating New Domain Bridges

To create a domain bridge for a non-software domain:

1. **Map all 11 primitives** to domain-specific concepts. Every primitive must have at least one concrete translation.

2. **Identify domain teams**. Who are the domain experts that will be dispatched at each phase? (Equivalent of the OS's 10 teams.)

3. **Define domain-specific audit criteria**. The 8 universal audit dimensions apply, but their measurement methods and evidence requirements change per domain.

4. **Create the bridge file** at `os/kernel/integration/domain_bridge_<domain>.md` following this file's format.

5. **Register the bridge** in the kernel manifest so the Controller Loop can load it based on the build request's domain classification.

**The kernel's value proposition**: Build the structural reasoning engine once. Reuse it across every domain by swapping the bridge. The audit backbone, confidence vectors, evolution ledger, reroute logic, and controller loop all work identically regardless of domain.

---

## Bridge Verification Checklist

Before a domain bridge is considered complete:

- [ ] All 11 primitives are mapped to domain concepts
- [ ] Each mapping has a responsible team or role identified
- [ ] Each mapping has translation rules that prevent ambiguity
- [ ] The audit dimensions' measurement methods are adapted for the domain
- [ ] The phase-to-team dispatch mapping is defined (see `kernel_os_bridge.md`)
- [ ] At least one example system has been traced through the bridge to verify completeness

---

*Domain Bridge v1.0 — Kernel Integration Layer*
