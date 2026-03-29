# Microservices Architecture Template

**Version:** 1.0
**Owner:** Architect Agent (Team 2)
**Last Updated:** 2026-03-29

---

## Purpose

This template provides the standard blueprint for microservices-based systems. It covers service decomposition, communication patterns, data management, deployment, and operational requirements. Use this template as the starting point for any system classified as microservices architecture.

---

## 1. Service Decomposition

### Bounded Context Identification

Before defining services, identify bounded contexts through Domain-Driven Design:

1. **Event Storming Session**: Map domain events, commands, aggregates, and policies
2. **Context Mapping**: Identify relationships between contexts (partnership, customer-supplier, conformist, anti-corruption layer, shared kernel, open host)
3. **Service Boundary Validation**:
   - Does this context have a distinct ubiquitous language?
   - Can it be owned by a single team?
   - Does it have its own data lifecycle?
   - Can it be deployed independently?

### Service Sizing Guidelines

- **Too large**: Multiple teams need to coordinate on deployments
- **Too small**: A single business operation requires 5+ synchronous inter-service calls
- **Right size**: One team owns it, it has a clear domain boundary, it can be deployed independently, and it has a meaningful data store

### Standard Service Structure

```
service-name/
  src/
    domain/           # Entities, value objects, domain events, domain services
    application/      # Use cases, command/query handlers, application services
    infrastructure/   # Database repos, HTTP clients, message publishers
    api/              # Controllers, gRPC handlers, event consumers
    config/           # Service configuration, DI wiring
  tests/
    unit/             # Domain and application logic tests
    integration/      # Infrastructure integration tests
    contract/         # Consumer-driven contract tests
  migrations/         # Database migration scripts
  Dockerfile
  docker-compose.yml  # Local development
  openapi.yaml        # API specification
  asyncapi.yaml       # Event specification (if applicable)
  README.md
```

---

## 2. Communication Patterns

### Synchronous Communication

**When to use:** Request-response patterns where the caller needs an immediate answer.

**Protocol Selection:**
| Use Case | Protocol | Rationale |
|----------|----------|-----------|
| Public APIs | REST (JSON) | Universal client support, tooling ecosystem |
| Internal service-to-service | gRPC | Performance, strong typing, streaming support |
| Complex queries | GraphQL | Client-driven data fetching, reduces over-fetching |

**Mandatory Requirements:**
- Timeouts on every call (default: 5 seconds, adjustable per endpoint)
- Circuit breakers wrapping every external service call
- Retry with exponential backoff and jitter for transient failures
- Bulkhead isolation (connection pool per downstream service)
- Request/response logging with trace context propagation

### Asynchronous Communication

**When to use:** Fire-and-forget, event notification, eventual consistency is acceptable.

**Pattern Selection:**
| Pattern | Use Case | Example |
|---------|----------|---------|
| Domain Events | Notify other contexts of state changes | `OrderPlaced`, `PaymentReceived` |
| Command Messages | Request another service to perform an action | `ProcessPayment`, `SendEmail` |
| Event Sourcing | Complete state history required | Financial transactions, audit trails |

**Message Broker Requirements:**
- At-least-once delivery guarantee
- Consumer idempotency (every consumer can safely process duplicates)
- Dead letter queue for poison messages
- Message ordering within partition/key
- Schema registry for event schema evolution
- Monitoring: consumer lag, DLQ depth, throughput

### API Gateway

**Responsibilities:**
- Request routing to appropriate service
- Authentication and token validation
- Rate limiting (per client, per endpoint)
- Request/response transformation
- TLS termination
- Request logging and tracing initiation

---

## 3. Data Management

### Database per Service (Mandatory)

Each service owns a private data store. No exceptions.

**Rules:**
- No service reads from another service's database
- Shared data is accessed through APIs or consumed through events
- Each service selects the database technology best suited to its access patterns
- Schema migrations are managed by the owning service only

### Data Consistency

**Strategy Selection:**
| Consistency Need | Pattern | Implementation |
|-----------------|---------|---------------|
| Within a service | ACID transactions | Database transactions |
| Across services (simple) | Saga - Choreography | Domain events + compensating actions |
| Across services (complex) | Saga - Orchestration | Saga orchestrator service |
| Read model sync | CQRS | Event-driven projections |

### Data Replication for Read Models

When a service needs data owned by another service for read-heavy queries:
1. Subscribe to events from the owning service
2. Build a local read-optimized projection
3. Accept eventual consistency (document the consistency window)
4. Implement projection rebuild capability

---

## 4. Service Discovery and Load Balancing

### Service Discovery
- **Kubernetes:** Use native DNS-based service discovery (`service-name.namespace.svc.cluster.local`)
- **Non-K8s:** Use Consul, etcd, or cloud-native service discovery (AWS Cloud Map, GCP Service Directory)

### Load Balancing
- **External traffic:** Application Load Balancer (ALB) or API gateway
- **Internal traffic:** Client-side load balancing (gRPC built-in) or service mesh (Istio, Linkerd)
- **Strategy:** Round-robin with health-check exclusion; weighted for canary deployments

---

## 5. Security

### Service-to-Service Authentication
- mTLS for all internal communication (service mesh provides this transparently)
- JWT tokens for propagating user identity across service boundaries
- Service accounts with least-privilege IAM roles per service

### API Security
- OAuth 2.0 / OpenID Connect for external APIs
- API key + rate limiting for machine-to-machine integrations
- Input validation at every service boundary (never trust upstream validation)

### Secrets Management
- All secrets in a vault (HashiCorp Vault, AWS Secrets Manager, GCP Secret Manager)
- Secrets injected at runtime, never baked into images or config files
- Automatic secret rotation with zero-downtime

---

## 6. Observability

### Logging Standard
```json
{
  "timestamp": "2026-03-29T10:15:30.123Z",
  "level": "INFO",
  "service": "order-service",
  "traceId": "abc123",
  "spanId": "def456",
  "message": "Order created",
  "orderId": "ord-789",
  "userId": "usr-012",
  "duration_ms": 45
}
```

### Metrics (RED + USE)
Every service exposes:
- Request rate, error rate, duration (RED) per endpoint
- CPU utilization, memory utilization, connection pool saturation (USE)
- Business metrics specific to the service domain

### Distributed Tracing
- W3C Trace Context propagation on all calls
- Span creation for: inbound requests, outbound HTTP/gRPC, database queries, message publish/consume
- Trace sampling: 100% for errors, 10% for normal traffic (adjustable)

### Health Endpoints
- `GET /health/live` — Liveness (is the process running?)
- `GET /health/ready` — Readiness (can it serve traffic?)
- `GET /health/startup` — Startup (has it finished initialization?)

---

## 7. Deployment

### Container Standards
- Multi-stage Dockerfile: build stage + minimal runtime image
- Non-root user in container
- Read-only root filesystem
- Resource limits defined (CPU, memory)
- Health check in Dockerfile

### Kubernetes Deployment
```yaml
# Every service includes:
- Deployment with rolling update strategy
- HorizontalPodAutoscaler (CPU/memory/custom metrics)
- PodDisruptionBudget (minAvailable: 1)
- Service (ClusterIP for internal, LoadBalancer for external)
- ConfigMap for non-secret configuration
- NetworkPolicy restricting ingress/egress
```

### Deployment Strategy
- **Default:** Rolling update (zero-downtime)
- **High-risk changes:** Canary deployment (route 5% traffic, monitor, then scale)
- **Database migrations:** Blue-green with backward-compatible migrations

---

## 8. Testing Strategy

### Test Pyramid per Service
| Level | Scope | Speed | Coverage Target |
|-------|-------|-------|----------------|
| Unit | Domain logic, use cases | <1s per test | 90%+ |
| Integration | Database, message broker, external APIs | <5s per test | All integration points |
| Contract | API contracts between services | <2s per test | All public APIs |
| E2E | Critical user journeys across services | <30s per test | Top 5 journeys |

### Contract Testing (Mandatory)
- Consumer-driven contract tests for all service-to-service APIs
- Provider verifies consumer contracts in CI pipeline
- Breaking a consumer contract blocks the provider's deployment
- Tools: Pact, Spring Cloud Contract, or equivalent

---

## 9. Failure Handling Checklist

- [ ] Every external call has a timeout configured
- [ ] Circuit breakers protect all downstream calls
- [ ] Retry logic uses exponential backoff with jitter
- [ ] Bulkhead isolation prevents resource exhaustion from one dependency
- [ ] Graceful degradation defined for every dependency failure
- [ ] Dead letter queues capture failed message processing
- [ ] Saga compensating actions defined for all distributed operations
- [ ] Health checks exclude unhealthy instances from load balancer
- [ ] Alerts fire on error rate increase, latency degradation, consumer lag

---

*This template is a starting point. Adapt to the specific system's requirements, scale, and team structure. Over-engineering a simple system is as harmful as under-engineering a complex one.*
