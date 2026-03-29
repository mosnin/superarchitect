# QUALITY GATES — Autonomous Quality Assurance

> "Quality gates are not checkpoints — they are continuous invariants that the system must maintain from the first artifact to the last."

---

## Gate Philosophy

A quality gate is an invariant, not a hurdle. The distinction matters: a checkpoint is something you pass through once and forget; an invariant is a property that must remain true at all times and is enforced continuously.

Every gate defined in this document is evaluated:
- After each team produces an output
- After remediation of any finding
- Before Commander synthesizes the final blueprint
- As part of the Reviewer's structured review process

Gates are binary: **PASS** or **FAIL**. There is no partial credit. A gate in FAIL state means the artifact is not ready to proceed.

### Why Continuous Invariants?

In a multi-agent pipeline, the most dangerous failure mode is not a single agent producing bad output — it is two agents producing individually acceptable output that, when combined, is inconsistent or incomplete. Continuous gates catch drift as it happens, not after synthesis when it is expensive to fix.

---

## Gate Types

### Completeness Gates
**Definition:** Verify that all required content is present.
**Failure signature:** Missing section, empty field, "TBD" in a critical position, unresolved placeholder.
**Responsible agent:** The team that produced the incomplete artifact.

### Consistency Gates
**Definition:** Verify that outputs from different teams agree with each other.
**Failure signature:** Technology named in architecture does not match infrastructure plan; entity named in API does not exist in data model; component count in architecture does not match team ownership list.
**Responsible agent:** The Reviewer, with remediation routed to the teams in conflict.

### Standards Gates
**Definition:** Verify that outputs follow the OS's documented standards (naming conventions, ADR format, blueprint section structure, etc.).
**Failure signature:** Non-standard naming, missing ADR for a major decision, incorrect severity labels in review output.
**Responsible agent:** The team that produced the non-standard artifact.

### Security Gates
**Definition:** Verify that security requirements are represented in design artifacts.
**Failure signature:** Public endpoint without auth, missing encryption spec for PII, threat model missing a named entry point, no audit logging defined.
**Responsible agent:** Security team, with review by Reviewer.

### Feasibility Gates
**Definition:** Verify that proposed designs are actually buildable with the stated team, timeline, and technology constraints.
**Failure signature:** Phase 1 scope requires 6 months of work but timeline says 6 weeks; design requires a technology the team has no experience with; infrastructure cost for MVP exceeds stated budget.
**Responsible agent:** DevOps team + Product team jointly.

---

## Gate Definitions

Each gate has: an ID, a name, the gate type, what is being checked, the failure condition, and the remediation route.

---

### GATE-001: Architecture Covers All Requirements
**Type:** Completeness
**Checks:** Every functional requirement in the requirements artifact is addressable by at least one named component in the architecture.
**Failure:** One or more requirements have no corresponding component in the architecture diagram or component list.
**Remediation:** Architect team adds the missing component(s) or explicitly notes how an existing component covers the requirement.

---

### GATE-002: All Public API Endpoints Have Security Defined
**Type:** Security
**Checks:** Every API endpoint defined by the Engineer team has an explicit authentication requirement (authenticated, public, or service-to-service) and an authorization rule.
**Failure:** Any endpoint is documented without a security annotation.
**Remediation:** Security team and Engineer team jointly add security specifications to undocumented endpoints.

---

### GATE-003: Data Model Covers All Entities Referenced in Requirements
**Type:** Completeness + Consistency
**Checks:** Every entity noun in the requirements document (user, workspace, project, order, product, etc.) has a corresponding table or document in the data model.
**Failure:** A requirement references an entity with no data model representation.
**Remediation:** Data team adds the missing entity with appropriate fields and relationships.

---

### GATE-004: NFRs Are Measurable
**Type:** Standards
**Checks:** Every non-functional requirement is expressed as a measurable, specific target — not a vague aspiration.
**Failure condition examples:**
- "The system should be fast" → FAIL
- "The system should be available" → FAIL
- "p99 API latency < 200ms under 10K concurrent users" → PASS
- "99.9% uptime (< 8.7 hours downtime/year), measured monthly" → PASS
**Remediation:** Product team rewrites vague NFRs into specific, measurable targets.

---

### GATE-005: Every Architecture Component Has an Owner Team
**Type:** Completeness
**Checks:** Every component in the architecture has a designated owner team (Architect, Engineer, Data, DevOps, Security) responsible for its specification and implementation plan.
**Failure:** Any component is listed without an owner.
**Remediation:** Commander assigns ownership based on component type; assignment is logged in the blueprint.

---

### GATE-006: Threat Model Covers All Entry Points
**Type:** Security
**Checks:** The security threat model addresses every external entry point identified in the architecture: all API endpoints, WebSocket connections, webhook receivers, admin interfaces, background job triggers, and data import paths.
**Failure:** An entry point exists in the architecture with no threat model entry.
**Remediation:** Security team extends the threat model to cover the missing entry point(s).

---

### GATE-007: CI/CD Pipeline Covers All Deployment Environments
**Type:** Completeness
**Checks:** The CI/CD pipeline definition includes stages for every environment described in the infrastructure plan (typically: development, staging, production; and for some systems: disaster recovery, preview environments).
**Failure:** An environment is defined in infrastructure but has no corresponding CI/CD pipeline stage.
**Remediation:** DevOps team adds the missing pipeline stage(s).

---

### GATE-008: Blueprint Is Self-Contained
**Type:** Completeness + Consistency
**Checks:** The blueprint contains no dangling references — every component, technology, entity, or service named in any section is either defined in the blueprint itself or in a referenced external document that is available.
**Failure:** A section references a component, decision, or document that does not exist in the blueprint or in a reachable reference.
**Remediation:** The team responsible for the referencing section either defines the referenced artifact inline or removes the reference.

---

### GATE-009: ADRs Exist for All Non-Default Technology Choices
**Type:** Standards
**Checks:** Every technology selection that deviates from the OS default stack has a corresponding Architecture Decision Record (ADR) documenting: the decision, the context, the options considered, the rationale for the selected choice, and the trade-offs accepted.
**Failure:** A non-default technology is used without an ADR.
**Remediation:** Architect team creates the missing ADR before the blueprint is finalized.

---

### GATE-010: Multi-Tenancy Isolation Is Explicitly Specified
**Type:** Security + Completeness
**Checks:** For any system identified as multi-tenant, the blueprint explicitly states the isolation model (schema-per-tenant, row-level security, or database-per-tenant) and provides evidence that the isolation model is enforced at the database and API layers.
**Failure:** System is multi-tenant but isolation model is not specified, or is mentioned vaguely without implementation detail.
**Remediation:** Data team and Security team jointly specify the isolation model and its enforcement mechanism.

---

### GATE-011: Phase 1 (MVP) Scope Is Explicitly Bounded
**Type:** Completeness + Feasibility
**Checks:** The implementation phases section explicitly identifies what is and is not in Phase 1 / MVP. Every feature in the system is assigned to a phase.
**Failure:** Features exist in the blueprint with no phase assignment, or "Phase 1" is so large it is effectively "build everything."
**Remediation:** Product team re-scopes Phase 1 to a deliverable, demonstrable MVP; remaining features are assigned to subsequent phases.

---

### GATE-012: Performance Targets Have a Load Model
**Type:** Feasibility
**Checks:** Every performance target (latency, throughput, uptime) is accompanied by a load model: the number of concurrent users, requests per second, or data volume that defines the target conditions.
**Failure:** A performance target exists without a load model (e.g., "p99 < 100ms" without specifying at what load).
**Remediation:** Product team adds the load model to each performance target.

---

### GATE-013: Authentication and Authorization Are Distinct and Both Specified
**Type:** Security
**Checks:** The security design separately and explicitly specifies:
1. Authentication: how users and services prove their identity
2. Authorization: how the system decides what an authenticated identity is permitted to do
**Failure:** Security section describes authentication but omits authorization model (or vice versa).
**Remediation:** Security team adds the missing specification.

---

### GATE-014: Database Schema Has Indexes for All Foreign Keys and Filter Columns
**Type:** Feasibility + Standards
**Checks:** The data model includes index definitions for:
- All foreign key columns
- All columns used in WHERE clauses in stated query patterns
- The tenant_id column (if multi-tenant)
- Any column listed as a sort key in high-frequency queries
**Failure:** A foreign key or high-frequency filter column has no index.
**Remediation:** Data team adds the missing indexes to the schema definition.

---

### GATE-015: Infrastructure Topology Matches Architecture Component Count
**Type:** Consistency
**Checks:** Every service, database, cache, and queue defined in the architecture diagram has a corresponding infrastructure resource defined in the infrastructure plan. No architecture component is "floating" without infrastructure.
**Failure:** Architecture names a service that has no corresponding infrastructure resource.
**Remediation:** DevOps team adds the missing infrastructure resources.

---

### GATE-016: Background Jobs Have Retry and Dead-Letter Queue Defined
**Type:** Completeness
**Checks:** Every background job or async worker defined in the system has an explicit retry policy (max attempts, backoff strategy) and a dead-letter queue or dead-letter handling strategy.
**Failure:** A background job exists without retry policy or DLQ.
**Remediation:** Engineer team adds retry and DLQ specifications.

---

### GATE-017: API Rate Limiting Is Specified for All Public Endpoints
**Type:** Security
**Checks:** The API design includes rate limiting specifications for all public-facing endpoints: the limit (requests per window), the window (per second/minute/hour), the key (per IP, per user, per API key), and the response on limit hit (429 with Retry-After).
**Failure:** Public endpoint exists without rate limit specification.
**Remediation:** Security team and Engineer team jointly add rate limit specifications.

---

### GATE-018: Observability Covers the Three Pillars
**Type:** Completeness
**Checks:** The infrastructure and DevOps plans include specifications for all three observability pillars:
1. Metrics: what is being measured, how, and what dashboards exist
2. Logging: structured log format, log levels, what events are logged
3. Tracing: distributed trace propagation, sampling strategy
**Failure:** Any of the three pillars is absent from the observability plan.
**Remediation:** DevOps team adds the missing observability specification.

---

### GATE-019: Disaster Recovery Strategy Is Defined
**Type:** Completeness
**Checks:** The blueprint includes a disaster recovery (DR) specification covering:
- Recovery Time Objective (RTO): maximum acceptable downtime
- Recovery Point Objective (RPO): maximum acceptable data loss
- Backup frequency and retention
- Restore procedure (at least at a high level)
- Who is responsible for initiating DR
**Failure:** DR section is absent or contains only RTO/RPO without procedure.
**Remediation:** DevOps team adds the DR specification.

---

### GATE-020: Secrets Are Never Hardcoded in Any Artifact
**Type:** Security
**Checks:** No secrets (API keys, database passwords, signing keys, OAuth credentials) appear as literal values in any artifact produced by the OS. Secrets are always referenced by environment variable name or secrets manager key path.
**Failure:** Any literal secret appears in any artifact.
**Remediation:** Immediate replacement with a reference. This gate has zero tolerance — it is always a BLOCK.

---

### GATE-021: API Responses Have Consistent Error Format
**Type:** Standards
**Checks:** The API design specifies a single, consistent error response format used across all endpoints (e.g., `{ "error": { "code": "...", "message": "...", "details": [...] } }`). All documented error scenarios use this format.
**Failure:** Multiple error formats exist across endpoints, or no error format is specified.
**Remediation:** Engineer team defines a canonical error format and applies it uniformly.

---

### GATE-022: Compliance Requirements Are Explicitly Mapped to Controls
**Type:** Security + Completeness
**Checks:** For any stated compliance requirement (SOC2, HIPAA, PCI DSS, GDPR), the blueprint maps each compliance control to a specific technical or procedural control in the design.
**Failure:** Compliance requirement is stated but the mapping to specific controls is absent.
**Remediation:** Security team produces the compliance control mapping.

---

### GATE-023: Test Strategy Covers All Critical Risk Areas
**Type:** Quality
**Checks:** The test strategy explicitly covers the highest-risk areas of the system as identified by the Reviewer: auth flows, financial transactions, multi-tenant isolation, data consistency under concurrent writes, and any other area flagged as high-risk.
**Failure:** A risk area identified in the security review or architecture review has no corresponding test coverage.
**Remediation:** QA team adds coverage for the uncovered risk area.

---

### GATE-024: External Dependencies Have Fallback Strategies
**Type:** Feasibility
**Checks:** Every external dependency (third-party API, payment processor, email service, identity provider) has a defined fallback strategy for when the dependency is unavailable — whether graceful degradation, queue-and-retry, or user-facing error with context.
**Failure:** External dependency is named without a fallback strategy.
**Remediation:** Engineer team or Architect team adds fallback strategy for each external dependency.

---

### GATE-025: Data Model Includes Audit Fields
**Type:** Standards
**Checks:** Every entity in the data model that represents a business object (not a pure join table) includes: `created_at`, `updated_at`, and where relevant, `created_by` and `updated_by` columns.
**Failure:** Business entity is missing standard audit fields.
**Remediation:** Data team adds the missing audit fields.

---

### GATE-026: Infrastructure Costs Are Estimated
**Type:** Feasibility
**Checks:** The infrastructure plan includes a rough cost estimate for the defined topology at the stated scale target (at least an order-of-magnitude estimate: <$1K/mo, $1K-$10K/mo, $10K-$100K/mo, >$100K/mo).
**Failure:** No cost estimate exists for the infrastructure plan.
**Remediation:** DevOps team adds a cost estimate using cloud provider pricing calculators.

---

### GATE-027: Handoff Package Is Complete
**Type:** Completeness
**Checks:** The handoff section of the blueprint includes all artifacts required for the selected handoff destination (Claude Code, GitHub, Linear, or manual) as defined in the OS handoff standards.
**Failure:** Handoff section is missing or does not include the required artifacts for the stated destination.
**Remediation:** Commander or the designated handoff team adds the missing artifacts.

---

### GATE-028: WebSocket or Streaming Architecture Has Connection Lifecycle Defined
**Type:** Completeness
**Checks:** For any system using WebSockets or persistent streaming connections: the connection lifecycle is defined (connect, authenticate, heartbeat, reconnect on drop, graceful close, abnormal close handling).
**Failure:** Streaming architecture exists without a connection lifecycle specification.
**Remediation:** Engineer team adds the connection lifecycle specification.

---

### GATE-029: Migrations Have a Rollback Plan
**Type:** Feasibility
**Checks:** Any schema migration defined in the data model section has a corresponding rollback plan — either an explicit down migration or a documented procedure for rolling back the change if it causes production issues.
**Failure:** A migration is defined without a rollback plan.
**Remediation:** Data team adds the rollback procedure for each migration.

---

### GATE-030: Success Criteria Are Verifiable
**Type:** Standards + Completeness
**Checks:** Every success criterion in the blueprint can be verified by a test, a metric, or an observable outcome. There are no success criteria that are subjective or unmeasurable.
**Failure:** A success criterion is present that cannot be verified (e.g., "users should find the product intuitive").
**Remediation:** Product team rewrites unverifiable success criteria as observable, measurable outcomes (e.g., "user task completion rate > 85% in usability testing with 10 participants").

---

### GATE-031: Encryption at Rest Is Specified for All PII
**Type:** Security
**Checks:** Every data entity containing personally identifiable information (PII) — names, emails, phone numbers, addresses, payment data, health data — has an explicit encryption-at-rest specification.
**Failure:** PII entity exists without an encryption-at-rest specification.
**Remediation:** Security team adds encryption specification for all PII-containing entities.

---

### GATE-032: API Pagination Is Defined for All List Endpoints
**Type:** Standards
**Checks:** Every list/collection endpoint in the API design specifies a pagination strategy (cursor-based, offset-based, or keyset) with documented maximum page size.
**Failure:** A list endpoint exists without pagination specification.
**Remediation:** Engineer team adds pagination specification to all list endpoints.

---

## Gate Failure Handling

When a gate fails during an Autopilot run:

### Immediate Actions
1. Gate failure is logged in the Run Log with timestamp, gate ID, and the specific artifact that caused the failure
2. The gate status in the quality gate dashboard is set to FAIL (red)
3. The responsible team is identified based on the gate definition
4. A remediation task is generated with specific, actionable instructions

### Automated Remediation
For gates where automated remediation is possible (GATE-025 audit fields, GATE-032 pagination), Autopilot attempts to add the missing content automatically and re-evaluates the gate.

### Human Escalation
If automated remediation is not possible and the gate is BLOCK-class (GATE-020, GATE-002, GATE-013), Autopilot pauses and surfaces the failure to the human operator.

### Re-Evaluation
After every remediation attempt, the gate is re-evaluated. If it passes, the Run Log records the resolution and the run continues. If it fails again after 3 attempts, it escalates to human.

---

## Gate Override

A gate can be bypassed only under the following conditions:

1. **The human operator explicitly requests the override** — no gate can be bypassed autonomously by Autopilot
2. **A documented justification is provided** — the override reason is logged in the Decision Log
3. **The override is time-bounded** — the override applies to this run only; the gate remains active for future runs
4. **A remediation plan is attached** — for BLOCK-class overrides, a specific plan to address the underlying issue must be attached

Override syntax:
```
GATE-OVERRIDE: GATE-[ID]
Reason: [Why this gate is being bypassed for this build]
Remediation plan: [How and when the underlying issue will be addressed]
Override authorized by: [Human operator identifier]
```

All overrides are permanently logged and cannot be deleted.

---

## Gate Dashboard

Every blueprint produced by the OS includes a Quality Gate Dashboard in its header:

```
QUALITY GATE STATUS
===================
Run ID: [UUID]
Evaluated: [ISO timestamp]
Total gates evaluated: 32

PASS  (27): 001 002 003 004 005 006 007 008 009 010 011 012 013 014
            015 016 017 018 019 020 021 022 023 024 025 026 027
FAIL   (0): —
SKIP   (5): 010 (not multi-tenant) 028 (no WebSocket) ...
OVERRIDE (0): —

Overall status: ALL GATES PASSED — BLUEPRINT CLEARED FOR HANDOFF
```

Gates that are not applicable to the current system type (e.g., GATE-010 multi-tenancy for a single-tenant system) are marked SKIP with the reason recorded.

A blueprint with any FAIL gates in non-skipped, non-overridden state cannot be delivered. This is enforced by Commander — synthesis is blocked until all applicable gates pass.
