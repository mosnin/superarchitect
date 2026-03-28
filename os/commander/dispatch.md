# SuperArchitect OS — Task Dispatch Logic

> *The Commander's routing brain. Given any task, this document determines which team(s) receive it, in what order, and with what constraints.*

---

## Purpose

This document defines the complete task dispatch system for the SuperArchitect OS Commander. It is loaded by the Commander after `COMMANDER.md` and used as the authoritative reference for all routing decisions. The Commander applies this logic for every task in every build.

---

## Task Classification Taxonomy

Every task receives a primary classification and optionally one or more secondary classifications. Classification determines which team is the primary owner and which teams are consulted.

### Primary Classifications

| Class | Code | Description | Primary Team |
|-------|------|-------------|--------------|
| Architecture | ARCH | System design, service decomposition, contract definition, ADR authoring, technology selection | Team 2 |
| Implementation — Backend | IMPL-BE | API development, business logic, data access, service communication | Team 3 |
| Implementation — Frontend | IMPL-FE | Web/mobile UI, design systems, accessibility, performance | Team 4 |
| Implementation — Data | IMPL-DATA | Pipelines, ML, analytics, data warehouse, feature stores | Team 5 |
| Infrastructure | INFRA | CI/CD, containerization, cloud infra, IaC, monitoring | Team 6 |
| Security | SEC | Threat modeling, auth/authz, compliance, vulnerability remediation | Team 7 |
| Quality Assurance | QA | Test strategy, test implementation, coverage analysis, performance testing | Team 8 |
| Product Design | PROD | Product brief, user stories, wireframes, UX flows, success metrics | Team 9 |
| Research | RES | Technology evaluation, competitive analysis, knowledge synthesis | Team 10 |
| Synthesis | SYN | Cross-team output integration, conflict resolution, build artifact assembly | Commander |
| Review | REV | Output validation, quality gate evaluation, standard compliance check | Commander + relevant team |
| Foundation | FOUND | OS maintenance, manifest updates, new team/workflow registration | Team 1 |

### Secondary Classifications (modifiers)

| Modifier | Meaning |
|----------|---------|
| `+SEC` | Security team must co-review this task's output |
| `+QA` | QA team must validate this task's output |
| `+PERF` | Performance considerations are critical for this task |
| `+COMPLIANCE` | Regulatory compliance requirements apply |
| `+DATA` | Data handling or privacy requirements apply |
| `+ML` | Machine learning components involved |
| `+CRITICAL` | Priority override — dispatch immediately |

**Example classifications**: `IMPL-BE +SEC +QA`, `ARCH +COMPLIANCE +DATA`, `INFRA +SEC`

---

## Routing Decision Tree

The Commander applies the following decision tree for every task:

```
START: Task received
│
├─ Is this a workflow invocation? (/build-system, /audit-system, /run-workflow)
│   ├─ YES → Load workflow file → Follow workflow phase ordering → EXIT
│   └─ NO → Continue to single-task routing below
│
├─ Classify the task (see taxonomy above)
│
├─ Is the primary class SYNTHESIS or REVIEW?
│   ├─ YES → Commander handles directly, no team dispatch needed → EXIT
│   └─ NO → Identify primary team from taxonomy
│
├─ Check team availability (is team at max_parallel_instances?)
│   ├─ AVAILABLE → Proceed to dependency check
│   └─ BUSY → Queue task; notify when slot opens
│
├─ Check input dependencies
│   ├─ ALL SATISFIED → Proceed to context packaging
│   └─ UNSATISFIED → Hold task; dispatch blocking tasks first
│
├─ Package context (prior phase outputs, relevant standards, patterns, decisions)
│
├─ Are secondary modifiers present?
│   ├─ +SEC → Co-dispatch Team 7 for security review of output
│   ├─ +QA → Co-dispatch Team 8 for QA validation of output
│   └─ other modifiers → note in TASK context, no additional dispatch needed
│
├─ Set priority level (see Priority Levels section)
│
└─ Send TASK message to primary team → AWAIT RESULT → Apply synthesis protocol
```

---

## Parallel vs. Sequential Dispatch Rules

### Dispatch Mode: PARALLEL

Tasks run in parallel when ALL of the following are true:
1. Neither task's output is an input to the other
2. Neither task has a data dependency on the other's output
3. Both tasks are in the same workflow phase
4. Combined active teams would not exceed `max_parallel_teams` (5) from `os/manifest.md`

**Parallel dispatch triggers a SYNC message** to all co-dispatched teams at start, so each team knows what others are producing in parallel.

### Dispatch Mode: SEQUENTIAL

Tasks run sequentially when ANY of the following is true:
1. Task B requires Task A's output as input
2. Task B's output would invalidate Task A's work if done in wrong order
3. Tasks access the same shared resource and could produce conflicts
4. One task is a REVIEW of another task's output
5. A quality gate separates the tasks

**Sequential dispatch sends one TASK message, awaits RESULT, validates, then sends the next TASK message.**

### Mixed Dispatch (Most Common)

Most phases use a mixed model: some parallel tracks with sequential gates between phases.

```
Example: Backend + Frontend + Data (parallel within Phase 3)
  But: QA (sequential after Phase 3 — requires their outputs)
  And: DevOps (mixed — starts with Architecture output, receives service
               specs from Backend as they complete)
```

---

## Dependency Graph Management

### Dependency Types

| Type | Symbol | Description |
|------|--------|-------------|
| Hard dependency | `→` | Task B cannot start until Task A is COMPLETE |
| Soft dependency | `⇢` | Task B can start but is enhanced by Task A's output |
| Review dependency | `✓→` | Task B waits for Commander's REVIEW of Task A |
| Parallel peer | `‖` | Tasks have no dependency on each other within a phase |

### Dependency Resolution Algorithm

```
When Task X is dispatched:
  1. Mark Task X as DISPATCHED
  2. Scan all queued tasks
  3. For each queued task T:
     a. Check T's requires_complete list
     b. If Task X is in T's requires_complete AND all others are COMPLETE:
        → Move T from QUEUED to READY_TO_DISPATCH
        → Dispatch T immediately (or at next dispatch cycle)

When a task moves to FAILED:
  1. Mark all downstream tasks as BLOCKED
  2. Attempt retry (up to max_retries_per_task)
  3. If retries exhausted → escalate to Commander synthesis for recovery
```

### Critical Path Identification

Before dispatching Phase 1, the Commander calculates the critical path:
```
Critical path = longest dependency chain from start to COMPLETE
All tasks NOT on the critical path = parallelizable slack
```

Tasks on the critical path receive HIGH priority automatically. Tasks with no downstream dependents receive MEDIUM priority.

---

## Load Balancing Heuristics

When multiple tasks are ready to dispatch simultaneously:

### Heuristic 1: Critical Path First
Tasks on the identified critical path dispatch before tasks with slack. This minimizes total build time.

### Heuristic 2: Security Always
If a security review task is ready alongside any other task, security dispatches first. The rationale: late security findings are more expensive to remediate than delayed non-security tasks.

### Heuristic 3: Upstream Unlocking Priority
Prefer tasks whose completion unblocks the most downstream tasks. If Task A unblocks 5 tasks and Task B unblocks 1, dispatch Task A first.

### Heuristic 4: Context Window Conservation
When approaching context window limits, prefer tasks that produce compact outputs (e.g., security review) over tasks that produce verbose outputs (e.g., full implementation) — unless the verbose output is on the critical path.

### Heuristic 5: Same-Team Batching
If a team has multiple ready tasks, batch them into a single TASK message where possible. This reduces overhead and gives the team the full picture for better decisions.

---

## Priority Levels

### CRITICAL
**Definition**: The build is blocked or a production system is at risk.
**Response**: Immediate dispatch, bypasses queue, may exceed parallel team limit by 1.
**Examples**: Security vulnerability in production system, build-blocking error in Architecture output, failed quality gate that must be remediated to proceed, human escalation response received.
**SLA**: Dispatch within 0 seconds of identification.

### HIGH
**Definition**: Task is on the critical path; delay increases total build time.
**Response**: Next to dispatch; top of queue.
**Examples**: Architecture blueprint (blocks all implementation), test coverage gate failure, API contract finalization.
**SLA**: Dispatch within the current dispatch cycle.

### MEDIUM
**Definition**: Important task with available slack; does not immediately block others.
**Response**: Normal queue ordering.
**Examples**: Frontend implementation (runs parallel to backend), DevOps pipeline setup, documentation authoring.
**SLA**: Dispatch within 2 dispatch cycles.

### LOW
**Definition**: Nice-to-have task; can be deferred without impacting build quality.
**Response**: Dispatched last; may be deferred to a follow-up build.
**Examples**: Advanced analytics dashboards, additional pattern documentation, non-critical tech debt remediation.
**SLA**: Dispatch when higher-priority tasks are complete.

---

## Example Dispatch Scenarios

### Scenario 1: New SaaS Product Build

**Input**: `/build-system — B2B project management SaaS with team workspaces, task tracking, time logging, billing, and API integrations`

```
WORKFLOW: build-saas.md

PHASE 1 — Discovery (parallel)
  DISPATCH [MEDIUM] → team-09-product
    TASK: Product brief for B2B project management SaaS
    Deliverables: User stories, user flows, design tokens, success metrics

  DISPATCH [MEDIUM] → team-10-research
    TASK: Research — SaaS project management space, technology recommendations
    Deliverables: Competitive analysis, tech stack recommendations, build-vs-buy analysis
                  for billing, time-tracking, and calendar integration

  DEPENDENCY: Phase 2 waits for both COMPLETE

PHASE 2 — Architecture (sequential)
  DISPATCH [HIGH] → team-02-architecture
    CONTEXT: Phase 1 outputs from Team 9 and Team 10
    TASK: Design system architecture for multi-tenant SaaS
    Deliverables: Blueprint, service catalog, data model, API contracts, ADRs
    CO-DISPATCH [+SEC]: team-07-security (threat model concurrent with architecture)

  DEPENDENCY: Phase 3 waits for Architecture COMPLETE

PHASE 3 — Implementation (parallel)
  DISPATCH [HIGH] → team-03-backend     [IMPL-BE +SEC +QA]
  DISPATCH [MEDIUM] → team-04-frontend  [IMPL-FE +QA]
  DISPATCH [MEDIUM] → team-05-data      [IMPL-DATA]  (billing analytics, time reports)
  DISPATCH [MEDIUM] → team-06-devops    [INFRA +SEC]

  NOTE: Backend on critical path (Frontend blocked on API contracts)
        DevOps can start with Architecture output; receives service specs as Backend completes

PHASE 4 — Quality + Security (parallel)
  DISPATCH [HIGH] → team-08-qa          [QA] (requires Phase 3 complete)
  DISPATCH [HIGH] → team-07-security    [SEC] (full security audit)

PHASE 5 — Synthesis
  Commander synthesizes all Phase 4 outputs
  Quality gates evaluated
  Final build artifact assembled
```

### Scenario 2: AI Platform Build

**Input**: `/build-system — Real-time AI fraud detection for payment processing. Sub-50ms inference. 10M transactions/day. PCI-DSS required.`

```
WORKFLOW: build-ai-platform.md

PHASE 1 — Research + Discovery (parallel)
  DISPATCH [HIGH] → team-10-research
    TASK: ML fraud detection approaches, streaming inference frameworks,
          PCI-DSS compliance requirements for AI systems
    NOTE: PCI-DSS flag — escalation likely needed for compliance scope

  DISPATCH [MEDIUM] → team-09-product
    TASK: Product brief — fraud detection use cases, alert workflows,
          analyst review interface, false positive management

  ESCALATION CHECK: PCI-DSS scope determination → escalate to human if unclear

PHASE 2 — Architecture (sequential)
  DISPATCH [HIGH] → team-02-architecture    [ARCH +COMPLIANCE +ML +PERF]
    CONTEXT: Research + product brief
    SPECIAL CONSTRAINTS: <50ms inference, 10M TPS, PCI-DSS scope
    CO-DISPATCH: team-07-security (PCI-DSS threat model)

PHASE 3 — Implementation (parallel, staggered)
  DISPATCH [HIGH] → team-05-data           [IMPL-DATA +ML +PERF]
    TASK: Feature engineering pipeline, model training infrastructure,
          real-time feature serving, model versioning
  DISPATCH [HIGH] → team-03-backend        [IMPL-BE +SEC +PERF]
    TASK: Inference API (<50ms gate), transaction intake, alert management API
  DISPATCH [MEDIUM] → team-04-frontend     [IMPL-FE]
    TASK: Analyst review dashboard, alert management UI
  DISPATCH [HIGH] → team-06-devops         [INFRA +SEC +PERF]
    TASK: GPU inference cluster, streaming infrastructure, PCI-compliant network

PHASE 4 — ML Evaluation + QA + Security (parallel)
  DISPATCH [HIGH] → team-05-data           [model evaluation, bias testing]
  DISPATCH [HIGH] → team-08-qa             [load testing, latency validation]
  DISPATCH [HIGH] → team-07-security       [PCI-DSS audit, pen test]

PHASE 5 — Compliance Review
  Commander synthesis + compliance sign-off
  ESCALATION if PCI-DSS gaps found
```

### Scenario 3: Enterprise Microservices

**Input**: `/build-system — Migrate legacy monolith ERP to microservices. 15 bounded domains. Multi-region. 99.99% SLA. SOC2 Type II required.`

```
WORKFLOW: build-enterprise.md

PHASE 1 — Analysis + Research (parallel)
  DISPATCH [HIGH] → team-02-architecture   [ARCH — domain analysis mode]
    TASK: Domain decomposition of ERP monolith into bounded contexts
    Deliverables: Domain map, migration strategy (strangler fig pattern), seam identification

  DISPATCH [HIGH] → team-10-research       [RES]
    TASK: Enterprise microservices patterns, service mesh evaluation,
          SOC2 requirements for distributed systems

  DISPATCH [MEDIUM] → team-07-security     [SEC +COMPLIANCE]
    TASK: SOC2 Type II control mapping for microservices architecture

PHASE 2 — Architecture (sequential)
  DISPATCH [CRITICAL] → team-02-architecture  [ARCH +COMPLIANCE]
    CONTEXT: Domain map, SOC2 controls, research report
    TASK: Full microservices architecture — service mesh, API gateway,
          event streaming, multi-region deployment topology

PHASES 3-7: [Implementation, Infrastructure, Testing, Security Audit, Synthesis]
  15 services dispatch in dependency order — complex graph managed by Commander
  Multi-region infrastructure dispatched as a parallel track
  SOC2 controls threaded through every implementation task via +COMPLIANCE modifier
```

### Scenario 4: Data Pipeline

**Input**: `/build-system — Real-time customer analytics pipeline. Sources: Postgres, Kafka, Salesforce. Target: Snowflake + Looker. SLA: < 5 min data latency.`

```
WORKFLOW: build-data-pipeline.md

PHASE 1 — Research + Architecture (parallel)
  DISPATCH [HIGH] → team-10-research
    TASK: ELT tools for Postgres+Kafka+Salesforce → Snowflake,
          dbt vs. Spark for transformation, Fivetran vs. custom connectors

  DISPATCH [HIGH] → team-02-architecture   [ARCH +DATA +PERF]
    TASK: Data pipeline architecture — ingestion, transformation, serving layers
    Constraint: <5 min end-to-end latency

PHASE 2 — Data Implementation (parallel)
  DISPATCH [HIGH] → team-05-data           [IMPL-DATA +PERF]
    TASK: Pipeline implementation — all connectors, transformations,
          Snowflake schema, dbt models, Looker LookML
  DISPATCH [MEDIUM] → team-06-devops       [INFRA]
    TASK: Orchestration infrastructure (Airflow/Prefect), monitoring

PHASE 3 — Quality + Security (parallel)
  DISPATCH [HIGH] → team-08-qa             [QA +DATA]
    TASK: Data quality validation, pipeline integrity tests, SLA tests
  DISPATCH [HIGH] → team-07-security       [SEC +DATA]
    TASK: PII data flow audit, access controls, encryption-at-rest

PHASE 4 — Synthesis
  Commander validates <5min latency gate
  Data lineage documentation assembled
```

---

## Dispatch State Machine

Each dispatched task moves through the following states. The Commander tracks all tasks.

```
QUEUED
  │ (dependencies met + team available)
  ▼
DISPATCHED (TASK message sent)
  │ (team begins work)
  ▼
ACTIVE (team acknowledged task)
  │ (team submits output)
  ▼
REVIEWING (Commander evaluating RESULT)
  │           │
  │ (pass)    │ (fail)
  ▼           ▼
COMPLETE    RETURNED (sent back with feedback)
              │
              │ (team revises)
              ▼
            REVIEWING (re-evaluation)
              │           │
              │ (pass)    │ (fail after max retries)
              ▼           ▼
            COMPLETE    FAILED → Commander escalation/recovery
```

---

*SuperArchitect OS v1.0 — os/commander/dispatch.md — Foundation Team (Team 1) — 2026-03-28*
