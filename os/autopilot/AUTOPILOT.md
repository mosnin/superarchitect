# AUTOPILOT — Fully Autonomous System Building

> "You give it a sentence. It gives you a production-ready system."

---

## What Is Autopilot?

Autopilot is the fully autonomous operating mode of SuperArchitect. When activated, the OS accepts a single natural-language request and — without further human input — activates all specialist teams, makes every architectural and technology decision, runs quality gates, resolves its own review findings, and produces a complete, production-grade system blueprint.

Think of it as the Tesla Autopilot of software architecture: the human sets the destination, the system handles every decision along the route.

### Capability Envelope

Autopilot is designed to handle:

- **Greenfield systems** of any scale and complexity — web platforms, APIs, data pipelines, ML systems, agent frameworks, mobile backends, CLI tools, internal tooling
- **Technology stack selection** for all layers — language, framework, database, cache, queue, infrastructure, CI/CD
- **Architecture pattern selection** — monolith, microservices, event-driven, CQRS, layered, hexagonal — based on the decision engine's rules
- **Security design** — threat modeling, auth design, compliance mapping, OWASP coverage
- **Infrastructure topology** — cloud provider, compute model (containers vs serverless), networking, storage, observability stack
- **Full blueprint production** — a complete System Blueprint ready for handoff to Claude Code, GitHub, or Linear

### Known Limitations

Autopilot will not:
- Make decisions that are **irreversible at the infrastructure level** (destroy production data, delete cloud resources, execute migrations against live databases)
- Assume compliance requirements that were not stated — it will flag the gap and ask
- Produce code — Autopilot produces blueprints, specs, and plans; code production is a downstream handoff to Claude Code or an engineer
- Override a human operator's explicit instruction, even if the decision engine would recommend otherwise
- Proceed past a BLOCK-severity quality gate finding after 3 remediation cycles without surfacing to a human

---

## Autopilot Activation

### Trigger Phrases

Autopilot activates when the user's request includes any of the following signals:
- "Build me a..."
- "Design a complete..."
- "I need a system that..."
- "Create a production-ready..."
- "Autopilot: [description]"
- Explicit `/autopilot` command

### Prerequisites

Before Autopilot proceeds, it confirms:

1. **A coherent system description exists** — even one sentence is sufficient; Autopilot will infer the rest and log its assumptions
2. **No conflicting explicit instructions** — if the user has stated contradictory requirements, Autopilot surfaces the conflict before proceeding
3. **Scope is bounded** — Autopilot identifies if a request is unbounded ("build me the next Google") and scopes it to a realistic MVP + roadmap

### Assumption Logging

Any information not provided by the user is decided by Autopilot and logged as an assumption in the blueprint's Decision Log. The human operator can review and override any assumption. Autopilot never silently assumes — every assumption is visible.

---

## Decision Authority

In Autopilot mode, the following categories of decisions are made autonomously by the Decision Engine:

### Technology Selections
- Programming language and framework per component
- Database technology per data store (transactional, analytical, cache, search, vector)
- Message broker / queue selection
- API protocol (REST, GraphQL, gRPC, WebSocket)
- Authentication library and identity provider
- Observability stack (metrics, logging, tracing)

All technology selections are recorded in the blueprint's ADR (Architecture Decision Record) section with rationale.

### Architecture Patterns
- Monolith vs microservices vs modular monolith (based on team size, scale target, complexity)
- Synchronous vs event-driven communication (based on coupling requirements and latency targets)
- Data access patterns (CQRS, repository pattern, active record)
- Caching topology (read-through, write-behind, cache-aside)
- Multi-tenancy model (schema isolation vs row-level security vs separate databases)

### Infrastructure Topology
- Cloud provider defaults (AWS preferred unless stated otherwise; see decision engine for rules)
- Compute model (Kubernetes for >3 services or >10K users; serverless for event-triggered, low-frequency workloads; VMs for legacy-adjacent or GPU workloads)
- Network topology (VPC design, subnet strategy, NAT, private endpoints)
- Storage tier selection (RDS for relational, DynamoDB for key-value at scale, S3 for object, EFS for shared filesystems)
- CDN strategy (CloudFront for global; skip for internal-only systems)

### Security Controls
- Auth model (JWT + refresh tokens for stateless APIs; session cookies for server-rendered; OAuth/OIDC for third-party identity)
- RBAC model design based on the system's stated user roles
- Encryption at rest for all PII and sensitive data (non-negotiable)
- WAF and rate limiting (applied by default to all public-facing APIs)
- Secret injection strategy (environment variables via secrets manager; no hardcoded secrets, ever)

### Implementation Approach
- Folder structure and module organization
- Error handling patterns
- Logging format and structured log schema
- API versioning strategy
- Test strategy and coverage targets

---

## Guardrails

Autopilot will never autonomously:

| Prohibited Action | Reason |
|-------------------|--------|
| Execute a database migration against a live system | Irreversible data risk |
| Delete cloud resources or infrastructure | Irreversible |
| Push to a production branch | Requires human sign-off |
| Deploy to a production environment | Requires human sign-off |
| Store, transmit, or log secrets or credentials | Security absolute |
| Select a compliance framework the user didn't mention | False assurance risk |
| Proceed past 3 unresolved BLOCK-severity review findings | Signals design ambiguity requiring human judgment |
| Change a previously approved ADR without flagging the change | Architecture drift risk |

---

## Confidence Thresholds

Autopilot scores its confidence in every significant autonomous decision (0-100%). The threshold determines whether it proceeds, logs a note, or pauses.

### High Confidence (>85%) — Proceed Autonomously
The decision engine has a clear rule match, the requirements are unambiguous, and there are no conflicting signals. Autopilot proceeds and logs the decision with rationale.

Example: "Build a REST API with 5 endpoints and 100 users" → PostgreSQL is the obvious database choice. Confidence: 95%. Proceed.

### Medium Confidence (60-85%) — Proceed with Decision Logged as Assumption
The decision engine has a rule match but the requirements have some ambiguity or the trade-offs are genuinely close. Autopilot proceeds but flags the decision in the Decision Log with an explicit note that the human operator should review.

Example: "Build a data platform for an analytics startup" → Kafka vs SQS is a genuine trade-off at early stage. Confidence: 72%. Proceed with Kafka (higher throughput ceiling), flag for review.

### Low Confidence (<60%) — Pause and Surface to Human
The decision engine cannot confidently choose between options, requirements are contradictory, or the decision has significant irreversible downstream effects. Autopilot pauses the run, surfaces the specific question to the human operator with a structured set of options, waits for input, then resumes.

Example: "Build a healthcare platform" — HIPAA compliance is uncertain without clarification. Confidence: 45%. Pause. "Does this system store or process PHI? If yes, HIPAA controls will be applied. If no, standard security posture will be used. Please confirm."

---

## Decision Logging

Every autonomous decision is recorded in the blueprint's **Autopilot Decision Log** section. Format:

```
DECISION: [Decision ID]
Timestamp: [ISO 8601]
Category: [Technology | Architecture | Infrastructure | Security | Quality]
Decision: [What was decided]
Rationale: [Why — which rule fired, what signals led here]
Confidence: [0-100%]
Alternatives Considered: [Other options that were evaluated]
Override: [Yes/No — was a human override applied?]
```

The Decision Log is a first-class section of every Autopilot-produced blueprint. It is the audit trail of the autonomous run.

---

## Autopilot Run Log

Every Autopilot run produces a structured run log. Format:

```
AUTOPILOT RUN LOG
=================
Run ID: [UUID]
Started: [ISO timestamp]
Request: "[Original user request]"
Status: [RUNNING | COMPLETED | PAUSED | FAILED]

PHASE LOG
---------
[HH:MM:SS] PHASE 1 — Requirements Analysis
  Team: Product + Researcher
  Status: COMPLETE
  Outputs: requirements.md, constraints.md, success-criteria.md
  Decisions: 3 logged
  Duration: [Xs]

[HH:MM:SS] PHASE 2 — Architecture Design
  Team: Architect
  Status: COMPLETE
  Outputs: system-architecture.md, component-diagram, adr-001 through adr-N
  Decisions: 12 logged
  Duration: [Xs]

[HH:MM:SS] PHASE 3 — Data Modeling
  Team: Data
  Status: COMPLETE
  Outputs: data-model.md, schema.sql, migration-strategy.md
  Decisions: 5 logged
  Duration: [Xs]

... [all phases] ...

[HH:MM:SS] REVIEW PASS
  Team: Reviewer
  Status: COMPLETE
  Findings: 0 BLOCK, 2 MUST, 5 SHOULD, 3 NIT, 4 PRAISE
  Verdict: APPROVED WITH CONDITIONS
  MUST items resolved: 2/2

[HH:MM:SS] SYNTHESIS
  Team: Commander
  Status: COMPLETE
  Output: system-blueprint.md

SUMMARY
-------
Total decisions made: [N]
Human pauses: [N]
Review cycles: [N]
Total duration: [Xs]
Final status: BLUEPRINT COMPLETE
```

---

## Recovery Protocol

When Autopilot encounters ambiguity or an unresolvable conflict mid-run:

### Step 1 — Classify the Blocker
- **Ambiguous requirement**: Requirements do not specify enough to make a confident decision
- **Conflicting requirements**: Two stated requirements contradict each other
- **Decision engine gap**: No rule covers this specific scenario
- **Review loop**: BLOCK finding cannot be resolved after 3 cycles

### Step 2 — Attempt Self-Resolution
For ambiguous requirements: check if a safe default exists (log it). For conflicting requirements: apply the more conservative option (log it). For decision engine gaps: fall back to OS default technology stack.

### Step 3 — Surface to Human (if self-resolution fails)
Autopilot pauses the run and presents:

```
AUTOPILOT PAUSED — INPUT REQUIRED
==================================
Run ID: [UUID]
Paused at: Phase [N] — [Phase Name]
Reason: [Clear description of why autopilot cannot proceed]

QUESTION
--------
[Specific, binary or limited-choice question — not open-ended]

OPTIONS
-------
A) [Option A] — [What this implies downstream]
B) [Option B] — [What this implies downstream]
C) Proceed with default ([default description])

Your response will be logged and the run will resume immediately.
```

### Step 4 — Resume
Upon receiving human input, Autopilot logs the response and resumes from the exact point of pause.

---

## Quality Assurance in Autopilot Mode

The Reviewer team operates in full autopilot mode as follows:

1. **Automated review:** After all team outputs are produced, the Reviewer runs all applicable checklists automatically against each artifact
2. **Finding classification:** All findings are classified by severity (BLOCK / MUST / SHOULD / NIT / PRAISE)
3. **Automatic remediation for MUST/SHOULD:** The Reviewer generates specific remediation instructions and routes them to the responsible team agent for resolution
4. **Human escalation for BLOCK (if unresolved after 3 cycles):** As described in Recovery Protocol above
5. **Quality gate execution:** All 30+ quality gates are evaluated and gate status is reported in the blueprint header
6. **Final approval:** The Reviewer issues a final verdict before Commander synthesizes the output

---

## Example Autopilot Run

### Request
```
Build a real-time collaborative document editor API
```

### Run Trace

**[00:00:00] Autopilot Activated**
Request received. Beginning requirements analysis phase.

**[00:00:01] Phase 1 — Requirements Inference (Product + Researcher)**

No explicit requirements provided. Autopilot infers from "real-time collaborative document editor API":
- Decision 001: System type = Collaborative SaaS API (confidence: 92%)
- Decision 002: Scale target = Medium (1K-10K concurrent users) — default for unstated scale (confidence: 70%, logged as assumption)
- Decision 003: Real-time requirement = YES, document synchronization required (confidence: 98%)
- Decision 004: Multi-user conflict resolution required — CRDT or OT algorithm needed (confidence: 95%)
- Decision 005: Auth required — authenticated users only (confidence: 99%)

Outputs produced:
- Functional requirements: document CRUD, real-time sync, presence awareness, version history, commenting, sharing/permissions
- NFRs: p99 sync latency < 100ms, 99.9% uptime, support 50 concurrent editors per document
- Success criteria: Full document round-trip sync in <100ms, conflict-free merges under concurrent edits

**[00:00:04] Phase 2 — Architecture Design (Architect)**

Decision engine fires:
- Rule: IF real-time sync required → SELECT WebSocket architecture (confidence: 97%)
- Rule: IF collaborative editing → SELECT CRDT over OT for conflict resolution (CRDT scales better operationally) (confidence: 82%)
- Rule: IF API-only (no frontend) → SELECT REST for document management + WebSocket for sync channel (confidence: 91%)
- Rule: IF scale < 100K concurrent → SELECT single-region deployment with horizontal scaling (confidence: 88%)

Architecture selected:
- Node.js / Fastify for HTTP API (performance, async I/O)
- WebSocket server (ws library) for real-time sync
- Yjs (CRDT library) for conflict-free document merging
- PostgreSQL for document persistence and metadata
- Redis for presence state and WebSocket session coordination across instances
- S3 for document snapshot storage (version history)

ADRs generated: ADR-001 (CRDT vs OT), ADR-002 (WebSocket vs SSE), ADR-003 (Redis for presence)

**[00:00:09] Phase 3 — Data Modeling (Data)**

Entities identified: Document, DocumentVersion, User, Collaborator, Presence, Comment, Permission
Schema produced with indexes, foreign keys, and RLS policies
Decision: Row-level security for document access control (no multi-tenancy required for API-only product)

**[00:00:13] Phase 4 — API Design (Engineer)**

REST endpoints: POST /documents, GET /documents/:id, PATCH /documents/:id, DELETE /documents/:id, POST /documents/:id/share, GET /documents/:id/versions, GET /documents/:id/comments, POST /documents/:id/comments

WebSocket protocol: CONNECT → AUTH → JOIN_DOCUMENT → SYNC_UPDATE → PRESENCE_UPDATE → LEAVE_DOCUMENT → DISCONNECT

Message schemas defined for all WebSocket events.

**[00:00:17] Phase 5 — Security Design (Security)**

- Auth: JWT (access token 15min, refresh token 30 days, stored in HttpOnly cookie)
- Authorization: Document-level permissions (owner, editor, commenter, viewer)
- Rate limiting: 100 req/min per user (REST), 1000 ops/min per document (WebSocket)
- Input validation: All document content sanitized (XSS prevention), size limits enforced (10MB per document)
- Audit log: All document access and modification events logged to append-only audit table
- Threat model: Covers broken auth, IDOR on document IDs (UUIDs, not sequential), WebSocket hijacking

**[00:00:21] Phase 6 — Infrastructure Design (DevOps)**

- AWS ECS Fargate (containerized, auto-scaling, no cluster management overhead)
- RDS PostgreSQL Multi-AZ (automated failover, daily snapshots)
- ElastiCache Redis (cluster mode for WebSocket session coordination)
- ALB with WebSocket support enabled (sticky sessions for WebSocket tier)
- S3 for document snapshots + CloudFront for static asset delivery
- CloudWatch for metrics + logs, X-Ray for distributed tracing

**[00:00:26] Phase 7 — Quality & Testing (QA)**

Test strategy:
- Unit: Yjs CRDT merge logic (100% coverage required), permission model (100% coverage required)
- Integration: WebSocket sync under concurrent load, document persistence round-trips
- E2E: Full collaborative session simulation (2+ concurrent editors, conflict resolution)
- Performance: WebSocket sync latency under 50 concurrent editors per document
- Chaos: Redis failure during active session (graceful degradation to polling fallback)

**[00:00:30] Phase 8 — Review (Reviewer)**

Review findings:
- PRAISE-001: CRDT selection over OT is the right long-term call — operationally superior
- MUST-001: WebSocket heartbeat / reconnection strategy is not specified — add to API design
- MUST-002: Document snapshot compaction strategy not defined — CRDT operation logs will grow unbounded
- SHOULD-001: Consider adding cursor position to presence state for richer UX
- NIT-001: "Collaborator" entity name conflicts with common usage — consider "DocumentMember"

Remediation triggered for MUST-001 and MUST-002. Engineer and Data teams resolve. Re-review passes.

Final verdict: APPROVED

**[00:00:38] Synthesis (Commander)**

Full System Blueprint assembled. Decision Log contains 23 decisions. All quality gates green. Blueprint delivered.

**Run Complete — Total Duration: 38 seconds**

---

## Kernel Mode Integration

Autopilot mode maps to the kernel's runtime modes:

| Autopilot Setting | Kernel Mode | Description |
|---|---|---|
| Full Auto (default) | Standard | 3 candidates, normal thresholds, autonomous routing |
| Deep Analysis | Search Heavy | 5+ candidates, deeper comparison, higher quality ceiling |
| Safety Critical | Conservative | Stricter thresholds, more evidence required, more escalation |
| Recovery | Overdrive | Triggered automatically when dimensions fall below 0.5 |

### Autopilot + Kernel Pipeline

In autopilot mode, the Commander runs the full 7-phase kernel pipeline without human intervention:
1. All phases execute autonomously
2. Rerouting happens automatically (up to 5 iterations)
3. Escalation triggers only at extreme thresholds (uncertainty > 0.85 + consequence = critical)
4. Every autonomous decision is logged in the evolution ledger
5. The canonical package captures the complete reasoning trail

---

*See `decision-engine.md` for the full rule set powering Autopilot decisions.*
*See `quality-gates.md` for the complete gate definitions enforced during every run.*
