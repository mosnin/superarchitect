# SuperArchitect OS — Inter-Agent Communication Protocols

> *The formal communication rules governing all message exchange between agents. Every agent must follow these protocols precisely. Non-conforming messages are rejected.*

---

## Purpose

This document defines the complete inter-agent communication protocol for the SuperArchitect OS. It specifies message formats, message types, handoff procedures, context passing, error handling, retry/recovery behavior, agent state machines, and output versioning.

All agents — Commander and all specialist teams — must send and receive messages that conform to this specification. Any agent receiving a non-conforming message must respond with a PROTOCOL_ERROR message rather than attempting to process the malformed input.

---

## Message Schema

Every message in the SuperArchitect OS has the following top-level structure. All fields are required unless marked optional.

```yaml
message:
  # Envelope (routing metadata)
  id: <string>             # Unique message ID: [workflow-id]-[phase]-[from]-[to]-[seq]
                           # Example: build-saas-p2-commander-team02-001
  from: <agent-id>         # Sender: "commander" | "team-NN-name" | "human"
  to: <agent-id>           # Recipient: "commander" | "team-NN-name" | "human" | "build-log"
  type: <message-type>     # One of the defined message types (see Message Types section)
  priority: <level>        # CRITICAL | HIGH | MEDIUM | LOW
  timestamp: <ISO-8601>    # When this message was created
  version: "1.0"           # Protocol version

  # Correlation
  reply_to: <message-id>   # [optional] ID of the message this is a response to
  thread_id: <string>      # Workflow+phase thread ID for grouping related messages
  sequence: <integer>      # Message sequence number within this thread

  # Content
  payload: <object>        # Message-type-specific content (see per-type schemas below)

  # Context (carried forward through the build)
  context:
    workflow_id: <string>          # Active workflow ID (e.g., "build-saas")
    phase: <string>                # Current workflow phase (e.g., "phase-2-architecture")
    build_id: <string>             # Unique ID for this entire build run
    prior_decisions: <list>        # ADR IDs that constrain this task
    active_standards: <list>       # Standards files the recipient must apply
    active_patterns: <list>        # Pattern files recommended for this task

  # Dependencies
  dependencies:
    requires_complete: <list>      # [optional] Task IDs that must be COMPLETE before action
    produces_for: <list>           # [optional] Task IDs that will consume this output
    blocks: <list>                 # [optional] Task IDs currently blocked by this message

  # Deadlines
  deadline: <ISO-8601 | "none">    # When this task must be complete
  sla_minutes: <integer | null>    # Maximum minutes for this task
```

---

## Message Types

### TASK
**Direction**: Commander → Team
**Purpose**: Assign a discrete unit of work to a specialist team. The team must not begin work until it receives a TASK message from the Commander.

```yaml
payload:
  objective: <string>             # One-sentence statement of what must be produced
  deliverables:                   # Explicit list of required output artifacts
    - artifact_id: <string>       # Short identifier for this artifact
      description: <string>       # What it is and what it must contain
      format: <string>            # Expected format: markdown | yaml | json | code | diagram
      required: <boolean>         # true = blocks RESULT; false = optional enrichment
  success_criteria: <string>      # How Commander will evaluate the output (specific, measurable)
  constraints:                    # Limits within which the team must operate
    - <constraint description>
  input_artifacts:                # Prior phase outputs provided as input
    - artifact_id: <string>
      source_team: <agent-id>
      content: <string | reference>
  guidance:                       # [optional] Specific direction, hints, or preferences
    - <guidance item>
```

**Example**:
```yaml
id: build-saas-p2-commander-team02-001
from: commander
to: team-02-architecture
type: TASK
priority: HIGH
timestamp: 2026-03-28T09:00:00Z
version: "1.0"
thread_id: build-saas-phase-2
sequence: 1

payload:
  objective: Design the complete system architecture for a multi-tenant B2B SaaS project management application
  deliverables:
    - artifact_id: architecture-blueprint
      description: Full system architecture document covering all services, their responsibilities, and communication patterns
      format: markdown
      required: true
    - artifact_id: service-catalog
      description: List of all microservices with names, responsibilities, owners, and interfaces
      format: yaml
      required: true
    - artifact_id: data-model
      description: Complete entity-relationship model with all tables, fields, and relationships
      format: markdown
      required: true
    - artifact_id: api-contracts
      description: OpenAPI 3.0 specifications for all service-to-service and client-facing APIs
      format: yaml
      required: true
    - artifact_id: adr-log
      description: Architecture Decision Records for all significant decisions made
      format: markdown
      required: true
  success_criteria: >
    Architecture supports 10,000+ concurrent users, is decomposed into independently
    deployable services, all inter-service contracts are explicit, multi-tenancy model
    is defined with clear isolation guarantees, and all ADRs document the trade-offs considered.
  constraints:
    - Multi-tenant: row-level security in PostgreSQL (tenant isolation model must be specified)
    - Cloud: AWS (EKS, RDS, ElastiCache, SQS, S3)
    - No vendor lock-in beyond major cloud services
    - Must support SaaS billing integration (Stripe)
  input_artifacts:
    - artifact_id: product-brief
      source_team: team-09-product
      content: "[product brief content]"
    - artifact_id: research-report
      source_team: team-10-research
      content: "[research report content]"

context:
  workflow_id: build-saas
  phase: phase-2-architecture
  build_id: build-20260328-001
  prior_decisions: []
  active_standards:
    - os/standards/api-design.md
    - os/standards/security-baseline.md
  active_patterns:
    - os/patterns/architectural/api-gateway.md
    - os/patterns/architectural/bulkhead.md

dependencies:
  requires_complete: [build-saas-p1-team09-001, build-saas-p1-team10-001]
  produces_for: [build-saas-p3-team03-001, build-saas-p3-team04-001, build-saas-p3-team05-001]
```

---

### RESULT
**Direction**: Team → Commander
**Purpose**: Deliver completed work output from a team to the Commander for review.

```yaml
payload:
  task_id: <string>               # ID of the TASK message this responds to
  status: COMPLETE | PARTIAL | FAILED
  artifacts:                      # Produced outputs
    - artifact_id: <string>       # Must match deliverable IDs from TASK
      content: <string>           # The actual artifact content
      format: <string>
      version: <string>           # Artifact version: timestamp-hash
  summary: <string>               # 2-5 sentence summary of what was produced and key decisions
  confidence: HIGH | MEDIUM | LOW # Team's confidence in the output quality
  self_review:                    # Results of the team's self-review pass
    standards_checked: <list>     # Standards files reviewed against
    issues_found: <list>          # Any issues identified (resolved or unresolved)
    resolved: <list>              # Issues fixed during self-review
    unresolved: <list>            # Issues team could not resolve — needs Commander attention
  handoff_notes: <string>         # What the next team(s) need to know
  unresolved_questions: <list>    # Questions that arose during execution, for Commander or human
  time_spent_estimate: <string>   # Approximate complexity (small | medium | large | xl)
```

---

### QUERY
**Direction**: Team → Commander, or Commander → Team
**Purpose**: Request specific information or clarification needed to complete a task. Queries must be precise and reference a specific gap in the available context.

```yaml
payload:
  subject: <string>               # One-line description of what is being asked
  question: <string>              # The full, precise question
  context_reference: <string>     # Which part of the task or prior output raised this question
  blocking: <boolean>             # true = team cannot proceed without answer
  answer_by: <ISO-8601 | "none">  # When the answer is needed
  options_considered:             # [optional] Options the team is considering
    - option: <string>
      trade_off: <string>
  my_recommendation: <string>     # [optional] Team's preferred answer if they have one
```

**Protocol rule**: A team may send at most 3 QUERY messages per TASK. More than 3 queries indicates the task was dispatched with insufficient context — the Commander must re-dispatch with a fuller context package.

---

### ESCALATION
**Direction**: Commander → Human (or Team → Commander for out-of-scope issues)
**Purpose**: Surface a decision or issue that requires human judgment or authority beyond the autonomous scope of the OS.

```yaml
payload:
  escalation_id: <string>         # Unique escalation identifier
  category: BUSINESS_TRADEOFF | COMPLIANCE | DESTRUCTIVE_OPERATION | AMBIGUITY | SCOPE_CHANGE | POLICY
  subject: <string>               # Short title of the escalation
  situation: <string>             # Full context — what was being built, what was found
  decision_required: <string>     # Exactly what the human must decide
  options:
    - id: <string>
      description: <string>
      pros: <list>
      cons: <list>
      estimated_cost_impact: <string | "none">
      estimated_timeline_impact: <string | "none">
  recommendation:
    preferred_option: <string>    # Commander's recommended option ID
    reasoning: <string>           # Why the Commander recommends this
  blocking:
    tasks_on_hold: <list>         # Task IDs blocked by this escalation
    estimated_unblock_delay: <string>   # How long the build is paused
  urgency: IMMEDIATE | TODAY | THIS_WEEK
```

---

### SYNC
**Direction**: Commander → Team(s) (broadcast or targeted)
**Purpose**: Deliver updated information, resolved conflicts, or status changes that an active team needs to know. SYNC messages do not require a response unless they include `requires_acknowledgment: true`.

```yaml
payload:
  sync_type: RESOLUTION | STATUS_UPDATE | CONTEXT_UPDATE | PHASE_COMPLETE | PHASE_START
  subject: <string>
  content: <string>               # The updated information
  requires_acknowledgment: <boolean>
  affected_artifacts: <list>      # Artifacts that need to be updated based on this SYNC
  effective_immediately: <boolean>
```

---

### BROADCAST
**Direction**: Commander → All active teams
**Purpose**: Communicate a significant OS-level event or decision that affects all active teams simultaneously (e.g., a global constraint change, a critical architectural pivot, a build pause).

```yaml
payload:
  event: GLOBAL_CONSTRAINT_CHANGE | ARCHITECTURAL_PIVOT | BUILD_PAUSE | BUILD_RESUME | CRITICAL_FINDING
  subject: <string>
  content: <string>               # Full description of the broadcast event
  affected_phases: <list>         # Phases/tasks impacted
  action_required: <boolean>      # Whether teams must update their work
  action_description: <string>    # If action required, what teams must do
```

---

### REVIEW_REQUEST
**Direction**: Team → Commander
**Purpose**: Request Commander review before final RESULT submission. Used when a team has completed work but is uncertain about a significant decision and wants Commander input before delivery.

```yaml
payload:
  task_id: <string>
  draft_artifacts: <list>         # Preliminary versions for review
  review_questions:               # Specific items requesting Commander feedback
    - <question>
  proposed_decisions:             # Decisions the team is about to make
    - decision: <string>
      rationale: <string>
      alternatives_considered: <list>
```

---

## Handoff Protocol

A handoff occurs when Team A's output becomes Team B's context. The Commander mediates all handoffs.

### Handoff Steps

1. **Team A submits RESULT** to Commander.
2. **Commander runs review** — completeness check, quality gate evaluation.
3. **Commander packages context** from Team A's RESULT:
   - Extract only the artifacts relevant to Team B's upcoming task
   - Compress verbose content (code bodies) into summaries where Team B doesn't need the full text
   - Annotate with Commander notes highlighting key decisions and constraints
4. **Commander sends TASK to Team B** with packaged context in `input_artifacts`.
5. **Commander sends SYNC to Team A** acknowledging RESULT and notifying if revisions needed.

### Context Packaging Rules

| Content Type | Handoff Treatment |
|--------------|------------------|
| Architecture blueprint | Full content — Team 3/4/5/6 need complete detail |
| API contracts | Full content — Teams 3/4 implement against these |
| Data model | Full content — Teams 3/5 implement against this |
| ADR log | Summary of decisions + full text of ADRs that constrain the next task |
| Research reports | Commander-written summary + key recommendations |
| Security threat model | Full content if next task is Security or DevOps; summary otherwise |
| Test plan | Summary for implementation teams; full content for QA |
| Implementation code | Never passed forward as context — summarize APIs and contracts only |

### Handoff Anti-Patterns (Prohibited)

- Passing raw code as context (use API summaries instead)
- Passing entire prior phase output without filtering (context window pollution)
- Skipping Commander mediation and having teams read each other's outputs directly
- Passing context without Commander annotation (teams lose the "why" behind decisions)

---

## Error Handling Protocol

### Error Categories

| Code | Name | Description | Commander Response |
|------|------|-------------|-------------------|
| `E001` | TASK_TIMEOUT | Team did not respond within `sla_minutes` | Re-dispatch with CRITICAL priority |
| `E002` | INCOMPLETE_OUTPUT | Required deliverable missing from RESULT | Return with specific request for missing item |
| `E003` | QUALITY_GATE_FAIL | Output fails one or more quality gates | Return with gate failure detail; request revision |
| `E004` | STANDARD_VIOLATION | Output violates a mandatory standard | Return with violated standard cited; require fix |
| `E005` | CONTEXT_INSUFFICIENT | Team could not complete task due to insufficient context | Re-dispatch with fuller context package |
| `E006` | DEPENDENCY_CONFLICT | Team's output conflicts with another team's output | Commander synthesis session; send SYNC with resolution |
| `E007` | SCOPE_EXCEEDED | Team produced output beyond the scope of the task | Accept within-scope portions; return out-of-scope portions |
| `E008` | PROTOCOL_VIOLATION | Message did not conform to protocol schema | Return PROTOCOL_ERROR; do not process message |
| `E009` | ESCALATION_TIMEOUT | Human did not respond to escalation within deadline | Re-send escalation with urgency upgrade |
| `E010` | CAPABILITY_GAP | Task requires capability the team does not have | Re-route to appropriate team or compose team response |

### Error Response Format

```yaml
from: commander
to: <originating-team>
type: TASK                        # Re-dispatch as a new TASK with error context
priority: HIGH
payload:
  objective: "[Same as original] — REVISION REQUIRED"
  error_code: <E-code>
  error_description: <specific description of what is wrong>
  revision_required: <exactly what the team must fix or add>
  original_task_id: <ID of the original TASK>
  accepted_artifacts: <list of artifacts that were accepted from the original submission>
  rejected_artifacts: <list with specific rejection reasons>
```

---

## Retry and Recovery Protocol

### Retry Logic

```
Task fails (RESULT with status: FAILED or quality gate failure)
  │
  ├─ Retry attempt 1: Re-dispatch with error context and additional guidance
  │   (priority elevated to HIGH)
  │
  ├─ Retry attempt 2: Re-dispatch with explicit direction on each failure point
  │   (priority: CRITICAL, Commander provides example or template for each failed item)
  │
  ├─ Retry attempt 3 (final): Re-dispatch with minimal scope reduction
  │   (Commander may reduce deliverable scope to unblock critical path)
  │
  └─ After max_retries_per_task exhausted:
      → CRITICAL escalation to Commander synthesis
      → Commander attempts direct synthesis from partial outputs
      → If synthesis fails → ESCALATION to human with detailed failure report
```

### Recovery Scenarios

**Scenario: Architecture RESULT fails quality gate (missing threat model)**
```
1. Commander sends E003 error back to team-02-architecture
2. Requests: "Add threat model for all external-facing services using STRIDE methodology"
3. Team revises and resubmits
4. Commander re-evaluates quality gate
```

**Scenario: Backend RESULT has 60% test coverage (gate requires 80%)**
```
1. Commander sends E003 error back to team-03-backend
2. Identifies specific modules below threshold
3. Requests: additional test cases for listed modules
4. Team fills gaps and resubmits coverage report
```

**Scenario: Context window exhausted mid-build**
```
1. Commander activates context compression
2. Compresses prior phase artifacts to summaries
3. Preserves full text only of artifacts directly needed by next team
4. Continues dispatch from last checkpoint
5. Full artifacts available in build artifact store (file system)
```

---

## Agent State Machine

Every agent in the OS exists in exactly one state at any time. The Commander maintains the state of all active agents.

```
                    ┌─────────────────────────────────────────────────┐
                    │                                                 │
          ┌─────────▼──────────┐                            ┌────────▼───────────┐
          │                    │   TASK received             │                    │
          │       IDLE         ├────────────────────────────►│    DISPATCHED      │
          │                    │                             │                    │
          └────────────────────┘                             └────────┬───────────┘
                    ▲                                                  │
                    │                                        Task acknowledged
                    │                                                  │
          ┌─────────┴──────────┐                            ┌─────────▼──────────┐
          │                    │                             │                    │
          │     COMPLETE       │◄────────────────────────── │      ACTIVE        │
          │                    │    RESULT submitted +       │                    │
          └─────────┬──────────┘    Commander review PASS   └─────────┬──────────┘
                    │                                                  │
                    │                                        RESULT submitted
                    │                                                  │
          ┌─────────▼──────────┐                            ┌─────────▼──────────┐
          │                    │   Review PASS               │                    │
          │  [Available for    │◄────────────────────────── │    REVIEWING       │
          │   next dispatch]   │                             │  (Commander eval)  │
          └────────────────────┘                             └─────────┬──────────┘
                                                                       │
                                                             Review FAIL / revision needed
                                                                       │
                                                            ┌──────────▼─────────┐
                                                            │                    │
                                                            │     RETURNED       │◄──── retry loop
                                                            │  (back to ACTIVE)  │
                                                            └──────────┬─────────┘
                                                                       │
                                                            max retries exhausted
                                                                       │
                                                            ┌──────────▼─────────┐
                                                            │                    │
                                                            │      FAILED        │
                                                            │                    │
                                                            └────────────────────┘
```

### State Definitions

| State | Meaning | Commander Action |
|-------|---------|-----------------|
| IDLE | Agent is available; no active task | Available for dispatch |
| DISPATCHED | TASK message sent; agent has not yet acknowledged | Monitor for acknowledgment; timeout after 30s |
| ACTIVE | Agent is working on the task | Monitor for RESULT or QUERY messages |
| REVIEWING | Commander is evaluating the RESULT | Block downstream dispatches until review complete |
| RETURNED | RESULT failed review; task sent back for revision | Track revision count against max_retries |
| COMPLETE | RESULT accepted; artifacts available for handoff | Package context for downstream teams |
| FAILED | Maximum retries exhausted or unrecoverable error | Escalate to Commander synthesis or human |

### State Transition Events

| From | To | Event |
|------|-----|-------|
| IDLE | DISPATCHED | Commander sends TASK message |
| DISPATCHED | ACTIVE | Team sends any message (implicit acknowledgment) |
| DISPATCHED | IDLE | TASK timeout — Commander re-queues |
| ACTIVE | REVIEWING | Team sends RESULT message |
| ACTIVE | ACTIVE | Team sends QUERY message (no state change) |
| REVIEWING | COMPLETE | Commander accepts RESULT (all gates pass) |
| REVIEWING | RETURNED | Commander rejects RESULT (sends back for revision) |
| RETURNED | ACTIVE | Team receives revised TASK and begins again |
| RETURNED | FAILED | Retry count exceeds max_retries_per_task |
| COMPLETE | IDLE | After build phase complete; team available for next dispatch |
| FAILED | IDLE | After Commander recovery; team reset for potential re-use |

---

## Versioning of Agent Outputs

Every artifact produced by any team is versioned using the following scheme:

### Version Identifier Format
```
[build-id]-[phase-id]-[team-id]-[artifact-id]-[timestamp]-[short-hash]

Example: build-20260328-001-p2-team02-arch-blueprint-20260328T0945-a1b2c3
```

### Version Storage

All versioned artifacts are stored in the build state during an active build. The Commander maintains a version manifest:

```yaml
build_artifact_manifest:
  build_id: build-20260328-001
  created: 2026-03-28T09:00:00Z
  last_updated: 2026-03-28T14:30:00Z
  artifacts:
    - id: arch-blueprint
      current_version: build-20260328-001-p2-team02-arch-blueprint-20260328T0945-a1b2c3
      versions:
        - version: build-20260328-001-p2-team02-arch-blueprint-20260328T0900-x9y8z7
          status: superseded
          reason: Revised after security threat model identified multi-tenant isolation gap
        - version: build-20260328-001-p2-team02-arch-blueprint-20260328T0945-a1b2c3
          status: current
```

### Version Immutability

Once an artifact is marked `status: current` and accepted by the Commander, it is immutable. If revisions are needed, a new version is created and the old one is marked `superseded`. The superseded version is retained for rollback capability.

### Downstream Impact Tracking

When a superseding version differs meaningfully from its predecessor, the Commander must:
1. Identify all downstream tasks that received the old version as context
2. Send a SYNC message to active downstream teams with the updated artifact
3. Flag completed downstream tasks for potential re-review if the change is material

---

## Protocol Compliance Checklist

Before sending any message, the sending agent should verify:

- [ ] `id` is unique and follows the naming convention
- [ ] `from` and `to` are valid agent IDs from the manifest
- [ ] `type` is one of the defined message types
- [ ] `priority` is set correctly relative to task urgency
- [ ] `timestamp` is present and current
- [ ] `payload` matches the schema for the given `type`
- [ ] `context.workflow_id` and `context.build_id` are present
- [ ] `dependencies.requires_complete` lists all true blockers
- [ ] All required artifacts from a RESULT are present
- [ ] Self-review section is complete for RESULT messages
- [ ] QUERY messages are specific and reference a context gap

---

*SuperArchitect OS v1.0 — os/commander/protocols.md — Foundation Team (Team 1) — 2026-03-28*
