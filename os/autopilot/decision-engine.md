# DECISION ENGINE — Autopilot Decision Rules and Heuristics

> "Every autonomous decision has a rule. Every rule has a reason. Every reason is logged."

---

## Overview

The Decision Engine is the brain of Autopilot. It is a structured rule system that maps system characteristics, requirements, and constraints to concrete technology, architecture, infrastructure, and quality decisions. When Autopilot makes a decision, it is because a rule fired — not because of a vague preference.

The engine is organized into four decision categories. Within each category, rules are ordered by specificity: more specific rules override general defaults.

---

## Decision Categories

### 1. Technology Decisions
Language, framework, database, message broker, caching, search, auth, observability.

### 2. Architecture Decisions
Pattern selection (monolith vs microservices), service decomposition, data access patterns, communication style (sync vs async), state management.

### 3. Infrastructure Decisions
Cloud provider, compute model (containers vs serverless vs VMs), networking, storage tiers, CDN, secrets management.

### 4. Quality Decisions
Test strategy, coverage targets, performance budgets, reliability targets, code quality standards.

---

## Decision Rules

Rules follow the format: **IF [condition] → [decision] (confidence)**

Conditions may be combined with AND / OR. More specific rules override general rules. Confidence scores reflect the engine's certainty that this rule is the right call for the stated condition.

---

### TECHNOLOGY — Language & Framework

**RULE-T001:** IF request specifies language explicitly → USE specified language (confidence: 100%)

**RULE-T002:** IF system type = API + team profile = Python → SELECT Python / FastAPI (confidence: 88%)

**RULE-T003:** IF system type = API + team profile = Node → SELECT Node.js / Fastify (confidence: 88%)

**RULE-T004:** IF system type = API + team profile = unspecified → SELECT Node.js / Fastify (confidence: 75%, flagged as assumption)

**RULE-T005:** IF system type = data pipeline → SELECT Python (confidence: 92%)

**RULE-T006:** IF system type = CLI tool → SELECT Go or Python based on distribution target (Go for single binary, Python for scripting ecosystem) (confidence: 84%)

**RULE-T007:** IF system requires real-time communication → SELECT Node.js (event loop model is optimal for WebSocket I/O) (confidence: 87%)

**RULE-T008:** IF system type = ML inference serving → SELECT Python / FastAPI + ONNX or PyTorch (confidence: 91%)

**RULE-T009:** IF performance requirement = ultra-low latency (<1ms) → SELECT Rust or Go (confidence: 89%)

**RULE-T010:** IF system type = mobile backend + team preference = Firebase → SELECT Firebase / Firestore (confidence: 83%)

---

### TECHNOLOGY — Database

**RULE-DB001:** IF data is relational AND no scale flag → SELECT PostgreSQL (confidence: 95%)

**RULE-DB002:** IF data is relational AND scale target > 1M rows/day writes → SELECT PostgreSQL with read replicas + connection pooler (PgBouncer) (confidence: 88%)

**RULE-DB003:** IF data is key-value AND read-heavy AND latency target < 10ms → SELECT Redis (confidence: 94%)

**RULE-DB004:** IF data volume > 1TB AND primary use = analytics → SELECT ClickHouse (columnar; superior for analytical queries) (confidence: 88%)

**RULE-DB005:** IF data model = document (deeply nested, schema-flexible) AND NOT requiring joins → SELECT MongoDB (confidence: 79%)

**RULE-DB006:** IF scale target > 10M records AND access pattern = single-key lookup → SELECT DynamoDB (confidence: 85%)

**RULE-DB007:** IF system requires full-text search → ADD Elasticsearch or OpenSearch as search layer (confidence: 90%)

**RULE-DB008:** IF system requires vector search / semantic similarity → SELECT pgvector (if data <10M) or Pinecone/Weaviate (if data >10M) (confidence: 84%)

**RULE-DB009:** IF compliance = financial data → REQUIRE PostgreSQL or equivalent ACID-compliant store (confidence: 97%)

**RULE-DB010:** IF time-series data → SELECT TimescaleDB (Postgres extension) for <1B rows, InfluxDB for larger (confidence: 86%)

---

### TECHNOLOGY — Message Broker & Queue

**RULE-Q001:** IF throughput target > 100K messages/sec OR event sourcing required → SELECT Apache Kafka (confidence: 92%)

**RULE-Q002:** IF use case = simple task queue AND scale < 100K msgs/day → SELECT AWS SQS (simpler ops, sufficient throughput) (confidence: 87%)

**RULE-Q003:** IF use case = background jobs + retry logic + scheduling → SELECT BullMQ (Redis-backed, excellent for Node) or Celery (Python) (confidence: 85%)

**RULE-Q004:** IF pub/sub at moderate scale → SELECT Redis Pub/Sub (if already using Redis) or SNS (confidence: 82%)

**RULE-Q005:** IF team size <= 3 → PREFER SQS over Kafka (operational overhead of Kafka is not justified at this team size) (confidence: 88%)

---

### TECHNOLOGY — Caching

**RULE-C001:** Default cache = Redis (confidence: 92%)

**RULE-C002:** IF cache data is immutable and globally distributed → ADD CloudFront or CDN cache layer (confidence: 90%)

**RULE-C003:** IF cache access pattern = session state → USE Redis with TTL (confidence: 95%)

**RULE-C004:** IF caching computed results for ML inference → USE Redis with configurable TTL (confidence: 88%)

---

### TECHNOLOGY — Observability

**RULE-O001:** Default metrics = Prometheus + Grafana (confidence: 90%)

**RULE-O002:** IF cloud = AWS → SELECT CloudWatch metrics + X-Ray tracing (lower operational overhead) (confidence: 85%)

**RULE-O003:** Default structured logging format = JSON (confidence: 98%)

**RULE-O004:** Default log aggregation = Loki (if self-hosted) or CloudWatch Logs / Datadog (if managed) (confidence: 82%)

**RULE-O005:** Default distributed tracing = OpenTelemetry (vendor-neutral) (confidence: 88%)

---

### ARCHITECTURE — Pattern Selection

**RULE-A001:** IF team size <= 3 AND scale target < 50K concurrent users → SELECT modular monolith (confidence: 91%)

**RULE-A002:** IF team size >= 5 AND domain = clearly decomposable bounded contexts → SELECT microservices (confidence: 84%)

**RULE-A003:** IF team size = 4-7 AND requirements are uncertain (early stage) → SELECT modular monolith with clear module boundaries (can extract services later) (confidence: 87%)

**RULE-A004:** IF system type = event-driven data pipeline → SELECT event-driven architecture with Kafka backbone (confidence: 93%)

**RULE-A005:** IF real-time sync required across multiple clients → SELECT event-driven + WebSocket architecture (confidence: 95%)

**RULE-A006:** IF read:write ratio > 10:1 → SELECT CQRS (separate read and write models) (confidence: 83%)

**RULE-A007:** IF audit trail required AND data immutability is critical → SELECT event sourcing (confidence: 80%)

**RULE-A008:** IF multiple distinct client types (web, mobile, third-party) → SELECT API Gateway pattern with BFF (Backend for Frontend) layer (confidence: 82%)

---

### ARCHITECTURE — Multi-Tenancy

**RULE-MT001:** IF multi-tenant AND compliance = SOC2/HIPAA/PCI AND < 1000 tenants → SELECT schema-per-tenant (strongest isolation, auditable) (confidence: 88%)

**RULE-MT002:** IF multi-tenant AND scale target > 10K tenants → SELECT row-level security with tenant_id (schema-per-tenant is operationally untenable at this scale) (confidence: 90%)

**RULE-MT003:** IF multi-tenant AND < 100 enterprise tenants AND strong data isolation contractually required → SELECT database-per-tenant (confidence: 85%)

**RULE-MT004:** IF multi-tenant → ALWAYS enforce tenant_id on every query, never trust client-provided tenant context (confidence: 99%)

---

### ARCHITECTURE — API Design

**RULE-API001:** IF API consumers = multiple client types with varying data needs → SELECT GraphQL (confidence: 84%)

**RULE-API002:** IF API = simple CRUD with well-defined resources AND public API → SELECT REST (confidence: 88%)

**RULE-API003:** IF API = internal service-to-service + low latency required → SELECT gRPC (confidence: 86%)

**RULE-API004:** IF real-time bidirectional communication → SELECT WebSocket (confidence: 96%)

**RULE-API005:** IF server-to-client push only (notifications, live feeds) → SELECT SSE (Server-Sent Events) as simpler alternative to WebSocket (confidence: 82%)

**RULE-API006:** IF API is public-facing → ALWAYS apply rate limiting (confidence: 99%)

**RULE-API007:** IF API versioning required → SELECT URI versioning (/v1/) for REST, schema versioning for GraphQL (confidence: 85%)

---

### INFRASTRUCTURE — Compute

**RULE-I001:** IF services > 3 OR concurrent users > 10K → SELECT Kubernetes (EKS/GKE) (confidence: 87%)

**RULE-I002:** IF services <= 3 AND scale is moderate AND ops simplicity valued → SELECT AWS ECS Fargate (managed containers, no cluster ops) (confidence: 85%)

**RULE-I003:** IF workload = event-triggered, low-frequency, stateless → SELECT serverless (AWS Lambda) (confidence: 89%)

**RULE-I004:** IF workload = ML training or GPU inference → SELECT GPU instances (EC2 P-series or G-series) (confidence: 95%)

**RULE-I005:** IF workload = batch data processing → SELECT serverless (Lambda + Step Functions) or ECS Fargate batch (confidence: 82%)

**RULE-I006:** IF system requires persistent connections (WebSocket, gRPC streams) → SELECT container-based compute (serverless is incompatible with persistent connections) (confidence: 98%)

---

### INFRASTRUCTURE — Cloud Provider

**RULE-CLOUD001:** IF cloud provider not specified → DEFAULT to AWS (widest service breadth, largest talent pool) (confidence: 80%, flagged as assumption)

**RULE-CLOUD002:** IF team is Google Cloud native OR Kubernetes-first → SELECT GCP (confidence: 78%)

**RULE-CLOUD003:** IF enterprise customer has Azure commitment → SELECT Azure (confidence: 90%)

**RULE-CLOUD004:** IF compliance = US Government → SELECT AWS GovCloud (confidence: 96%)

---

### INFRASTRUCTURE — Security Infrastructure

**RULE-SEC001:** IF system is public-facing → ADD WAF (AWS WAF or Cloudflare) (confidence: 93%)

**RULE-SEC002:** IF compliance = PCI DSS → REQUIRE tokenization + audit logs + encryption at rest + quarterly pen test plan (confidence: 98%)

**RULE-SEC003:** IF compliance = HIPAA → REQUIRE PHI encryption at rest + in transit + BAA with cloud provider + audit logs (confidence: 99%)

**RULE-SEC004:** IF compliance = SOC2 → REQUIRE audit logging + access controls + change management + incident response plan (confidence: 97%)

**RULE-SEC005:** IF secrets management → USE AWS Secrets Manager or HashiCorp Vault (never environment variables in code) (confidence: 96%)

**RULE-SEC006:** IF public API → REQUIRE DDoS protection (Shield Standard minimum, Shield Advanced for critical) (confidence: 91%)

---

### QUALITY — Testing

**RULE-QA001:** IF system type = API → REQUIRE integration tests covering all happy paths + top 5 error paths (confidence: 95%)

**RULE-QA002:** IF business logic is complex → REQUIRE unit test coverage >= 80% on business logic modules (confidence: 92%)

**RULE-QA003:** IF system handles financial transactions → REQUIRE 100% unit test coverage on transaction logic (confidence: 98%)

**RULE-QA004:** IF real-time sync → REQUIRE concurrent load test simulating max stated concurrent users (confidence: 94%)

**RULE-QA005:** IF deployment = multi-environment → REQUIRE E2E tests against staging before production deploy (confidence: 93%)

**RULE-QA006:** IF public API → REQUIRE contract tests (consumer-driven contracts) (confidence: 82%)

---

## Technology Default Stack

The OS's default technology choices when no specific constraint overrides them.

| Category | Default Choice | Reasoning |
|----------|---------------|-----------|
| API Framework | Node.js / Fastify | High throughput, async I/O, excellent TypeScript support, minimal overhead |
| Type System | TypeScript | Type safety catches entire classes of runtime bugs |
| ORM | Prisma (Node) / SQLAlchemy (Python) | Type-safe, migration-integrated, excellent DX |
| Relational DB | PostgreSQL 16 | ACID, JSONB, RLS, PostGIS, pgvector — most versatile open-source RDBMS |
| Cache | Redis 7 | Universal, fast, supports multiple data structures |
| Message Queue | SQS (low scale) / Kafka (high scale) | SQS for simplicity; Kafka when throughput or log retention matters |
| Search | Elasticsearch / OpenSearch | Proven, feature-rich, good managed options |
| Object Storage | AWS S3 | Industry standard, near-infinite scale, 11 nines durability |
| Container Registry | Amazon ECR | Integrates natively with ECS/EKS |
| CI/CD | GitHub Actions | Lowest friction for most teams; integrates with existing code hosting |
| Infrastructure as Code | Terraform | Multi-cloud, large module ecosystem, state management |
| Secrets | AWS Secrets Manager | Managed rotation, fine-grained IAM, audit trail |
| Metrics | Prometheus + Grafana | Industry standard; exporters exist for everything |
| Tracing | OpenTelemetry → Jaeger | Vendor-neutral; can redirect to Datadog/Honeycomb later |
| Logging | Structured JSON → CloudWatch / Loki | Queryable, parseable, compatible with all aggregation tools |
| Auth (service-to-service) | mTLS or signed JWTs | Zero-trust; no implicit trust between services |
| Auth (user-facing) | JWT + refresh tokens (HttpOnly cookie) | Stateless, scalable, secure against XSS when cookie-stored |
| Identity Provider | Auth0 or Cognito | Avoid building auth from scratch; delegate to proven IdP |

---

## Stack Override Protocol

Any default can be overridden at the blueprint level. Overrides must be:

1. **Explicitly stated** — in the build request or in a user instruction
2. **ADR-documented** — every override generates an Architecture Decision Record explaining why the default was not used
3. **Consistency-checked** — the Reviewer validates that the override does not conflict with other decisions

Override syntax in a build request:
```
OVERRIDE: database = MySQL (existing infrastructure constraint)
OVERRIDE: cloud = Azure (enterprise agreement)
OVERRIDE: framework = Django (team expertise)
```

Overrides are logged in the Decision Log with category `OVERRIDE` and the stated reason.

---

## Decision Conflict Resolution

When two rules fire and produce conflicting recommendations:

### Step 1 — Apply Specificity
More specific rules win. RULE-DB009 ("financial data → PostgreSQL") overrides RULE-DB006 ("key-value pattern → DynamoDB") because it addresses a compliance constraint, which is more specific than an access pattern preference.

### Step 2 — Apply Constraint Hierarchy
Constraints are ranked:
1. **Compliance / legal constraints** (highest priority — non-negotiable)
2. **Explicit user instructions**
3. **Scale / performance requirements**
4. **Operational constraints** (team size, existing infrastructure)
5. **Cost optimization**
6. **Default preferences** (lowest priority)

### Step 3 — Log the Conflict
If two rules at the same specificity level conflict, the conflict is logged in the Decision Log and the higher-priority constraint wins. The losing rule's recommendation is recorded as an "alternative considered."

### Step 4 — Surface if Unresolvable
If a conflict cannot be resolved by the hierarchy (two compliance constraints that genuinely contradict), Autopilot pauses and surfaces the conflict to the human operator with both options and their implications.

---

## Decision Confidence Scoring

Confidence is calculated as a weighted composite of four factors:

### Factor 1 — Rule Match Strength (weight: 40%)
- Exact rule match (condition fully met): 100
- Partial rule match (condition partially met): 60
- Default/fallback rule: 40
- No rule match, heuristic applied: 20

### Factor 2 — Requirement Clarity (weight: 30%)
- Requirement explicitly stated: 100
- Requirement strongly implied by context: 75
- Requirement inferred from system type: 50
- Requirement assumed (no signal): 25

### Factor 3 — Decision Reversibility (weight: 20%)
- Easily reversible (config change): 100
- Moderately reversible (refactor required): 70
- Difficult to reverse (data migration required): 40
- Effectively irreversible (architectural): 15

### Factor 4 — Historical Pattern Match (weight: 10%)
- Matches well-established industry pattern: 100
- Matches common but not universal pattern: 70
- Novel or uncommon pattern: 40

### Confidence Formula
```
confidence = (rule_match * 0.40) + (req_clarity * 0.30) + (reversibility * 0.20) + (pattern_match * 0.10)
```

Scores are rounded to the nearest integer. Scores below 60 trigger a human pause as described in the Autopilot confidence threshold policy.

---

*This decision engine is a living document. New rules are added when repeated manual decisions reveal a gap. Rules are updated when post-mortems identify incorrect confidence calibration.*
