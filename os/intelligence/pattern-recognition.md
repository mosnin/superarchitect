# Pattern Recognition Framework — SuperArchitect OS

**Version:** 1.0
**Owner:** Intelligence Layer
**Last Updated:** 2026-03-29

---

## Purpose

This framework defines how the Intelligence Layer identifies, matches, and applies patterns during system builds and audits. Pattern recognition is the core mechanism by which the OS leverages accumulated engineering knowledge to produce better outcomes faster.

---

## 1. Pattern Matching Process

### 1.1 Input Analysis

When analyzing a system (new build or audit), extract these features:

**Domain Features:**
- Business domain (e-commerce, fintech, healthcare, social, productivity)
- Regulatory requirements (GDPR, HIPAA, PCI-DSS, SOC2)
- User types (consumers, businesses, developers, internal)
- Transaction characteristics (read-heavy, write-heavy, mixed)

**Scale Features:**
- Expected user count (now and in 12 months)
- Data volume (GB, TB, PB)
- Request rate (requests per second)
- Latency requirements (interactive, near-real-time, batch)
- Availability requirements (99.9%, 99.99%)

**Team Features:**
- Team size and structure
- Technical expertise and technology familiarity
- Operational maturity (can they manage Kubernetes? Event streaming?)
- Deployment cadence expectations

**Integration Features:**
- Number of external integrations
- Integration patterns needed (sync, async, batch)
- Legacy system constraints
- Third-party API dependencies

### 1.2 Pattern Catalog Scan

Match extracted features against the pattern catalog (`os/teams/architect/patterns.md`):

```
For each pattern in catalog:
  1. Check preconditions (does the context match?)
  2. Check forces (are the driving forces present?)
  3. Check constraints (does the team/scale support this?)
  4. Calculate fit score (0-100)
  5. Identify conflicts with other matched patterns
```

### 1.3 Pattern Combination Validation

Verify that selected patterns work together:

**Compatible Combinations:**
- Hexagonal Architecture + CQRS + Event Sourcing
- Microservices + Database per Service + Saga Pattern
- Modular Monolith + Vertical Slice + In-Process Events
- Event-Driven + Transactional Outbox + CDC

**Incompatible Combinations:**
- Event Sourcing + Shared Database (contradicts data ownership)
- Microservices + Shared Mutable State (defeats independence)
- CQRS + Simple CRUD (overhead without benefit)
- Distributed Transactions + Microservices (fragile and slow)

---

## 2. Architecture Pattern Recognition

### 2.1 Signals for Monolith

| Signal | Strength |
|--------|----------|
| Single team (1-8 people) | Strong |
| Domain boundaries unclear | Strong |
| MVP or early-stage product | Strong |
| Strong consistency needed everywhere | Moderate |
| Low operational maturity | Strong |
| Simple deployment requirements | Moderate |

**Confidence threshold:** 3+ strong signals -> recommend monolith

### 2.2 Signals for Microservices

| Signal | Strength |
|--------|----------|
| Multiple independent teams | Strong |
| Different scaling requirements per domain | Strong |
| Independent deployment critical | Strong |
| Technology diversity needed | Moderate |
| Domain boundaries well-understood | Strong |
| High operational maturity | Moderate |

**Confidence threshold:** 3+ strong signals including "domain boundaries well-understood"

### 2.3 Signals for Event-Driven

| Signal | Strength |
|--------|----------|
| Multiple consumers for same events | Strong |
| Temporal decoupling needed | Strong |
| Real-time processing required | Strong |
| Audit trail requirements | Moderate |
| Integration with many external systems | Moderate |
| Eventual consistency acceptable | Strong |

**Confidence threshold:** 2+ strong signals including "eventual consistency acceptable"

### 2.4 Signals for CQRS

| Signal | Strength |
|--------|----------|
| Read/write ratio > 10:1 | Strong |
| Complex read queries (joins, aggregations) | Moderate |
| Simple write operations | Moderate |
| Different scaling needs for reads vs writes | Strong |
| Multiple read model shapes needed | Strong |

### 2.5 Signals for Event Sourcing

| Signal | Strength |
|--------|----------|
| Full audit trail required (regulatory) | Strong |
| Temporal queries needed | Strong |
| Business domain thinks in events naturally | Moderate |
| Need to reconstruct past state | Strong |
| Team has event sourcing experience | Moderate (absence is a warning) |

---

## 3. Anti-Pattern Detection

### 3.1 Code-Level Anti-Patterns

**Detection: God Object**
```
Indicators:
- File/class > 500 lines
- > 20 methods/functions in a single class
- > 10 dependencies imported
- High fan-in (many other files depend on this one)
Confidence: HIGH when 3+ indicators present
```

**Detection: N+1 Queries**
```
Indicators:
- Loop containing database query call
- ORM lazy loading in a list context
- Query count scales with result set size
- Performance degrades linearly with data volume
Confidence: HIGH when ORM + loop detected, MEDIUM on performance pattern alone
```

**Detection: Distributed Monolith**
```
Indicators:
- Multiple services sharing a database
- Synchronous call chains > 3 services deep
- Services that must deploy together
- Shared libraries with domain logic (not just utilities)
- Single team owning all services
Confidence: HIGH when shared database + sync chains detected
```

**Detection: Spaghetti Architecture**
```
Indicators:
- Circular dependencies between modules
- No clear layering (any module calls any other)
- Business logic in controllers/handlers
- Database queries in UI code
- Configuration scattered across codebase
Confidence: HIGH when circular dependencies + business logic in wrong layer
```

### 3.2 Infrastructure Anti-Patterns

**Detection: Snowflake Infrastructure**
```
Indicators:
- No IaC files (Terraform, Pulumi, CDK)
- Manual setup instructions in README
- Environment-specific config hardcoded
- "Works on my machine" deployment
Confidence: HIGH when no IaC files found
```

**Detection: Alert Fatigue**
```
Indicators:
- > 50 alert rules defined
- > 30% of alerts auto-resolve without action
- Alert acknowledgement time increasing over time
- Same alert fires repeatedly without resolution
Confidence: MEDIUM (requires operational data to confirm)
```

### 3.3 Security Anti-Patterns

**Detection: Secrets in Code**
```
Indicators:
- Strings matching API key patterns (32+ hex/base64 chars)
- Environment variables with names like SECRET, PASSWORD, TOKEN
- Connection strings with embedded credentials
- .env files committed to version control
Confidence: HIGH for pattern matches, verify with entropy analysis
```

**Detection: Missing Auth**
```
Indicators:
- API endpoints without authentication middleware
- Routes defined without authorization checks
- Public endpoints that mutate data
- Admin endpoints accessible without elevated permissions
Confidence: HIGH when data-mutating endpoints lack auth middleware
```

---

## 4. Technology Pattern Matching

### 4.1 Database Selection Heuristics

| Workload Pattern | Recommended Database | Rationale |
|-----------------|---------------------|-----------|
| Transactional, relational data | PostgreSQL | ACID, extensible, excellent ecosystem |
| Document-oriented, flexible schema | MongoDB or PostgreSQL JSONB | Schema flexibility with query power |
| High-throughput key-value | Redis, DynamoDB | Sub-millisecond latency at scale |
| Time-series data | TimescaleDB, InfluxDB | Optimized for time-based queries |
| Graph relationships | Neo4j, Neptune | Native graph traversal |
| Full-text search | Elasticsearch, Typesense | Inverted index, relevance scoring |
| Analytical / OLAP | ClickHouse, BigQuery | Columnar storage, fast aggregations |

### 4.2 Message Broker Selection Heuristics

| Workload Pattern | Recommended Broker | Rationale |
|-----------------|-------------------|-----------|
| High-throughput event streaming | Kafka | Append-only log, replay, high throughput |
| Task queues with routing | RabbitMQ | Flexible routing, mature, reliable |
| Serverless / cloud-native | SQS/SNS, Pub/Sub | Managed, scalable, low ops overhead |
| Low-latency messaging | NATS | Minimal overhead, cloud-native |

### 4.3 Framework Selection Heuristics

| Context | Recommended Approach |
|---------|---------------------|
| API-first backend, TypeScript team | Node.js + Fastify/Express + Prisma |
| High-performance backend, strict typing | Go + Chi/Gin + sqlc |
| Rapid development, Python team | Python + FastAPI + SQLAlchemy |
| Enterprise Java team | Java + Spring Boot + Hibernate |
| Systems programming, safety-critical | Rust + Actix/Axum |

---

## 5. Pattern Application Protocol

When the Intelligence Layer identifies applicable patterns, it follows this protocol:

1. **Present patterns to Architect Agent** with fit scores and rationale
2. **Architect Agent validates** against specific system requirements
3. **Architect Agent selects** the pattern combination
4. **Intelligence Layer checks** for incompatible combinations
5. **Architect Agent documents** the choice in an ADR
6. **Intelligence Layer records** the application for future learning

---

## 6. Continuous Improvement

After each build, the Intelligence Layer updates pattern effectiveness data:

```yaml
pattern_application:
  pattern: [name]
  context: [features that triggered the match]
  outcome: EFFECTIVE | PARTIALLY_EFFECTIVE | INEFFECTIVE
  notes: [what worked, what did not, what was learned]
  recommendation: [keep | modify | remove from heuristics]
```

This data feeds back into the pattern matching engine, improving accuracy over time.

---

*Pattern recognition is not about applying templates blindly. It is about recognizing the forces at play in a specific context and matching them to proven solutions. The intelligence is in the matching, not the patterns themselves.*
