# System Blueprint Protocol (SBP)

> The universal output format of the SuperArchitect OS — a complete, structured, machine-readable and human-readable system plan that any AI system, CI/CD pipeline, or engineering team can consume and execute without ambiguity.

---

## What is a System Blueprint?

A System Blueprint is the final, authoritative artifact produced by the SuperArchitect OS at the end of every build workflow. It is not a diagram, not a document, and not a backlog. It is all three simultaneously — and more.

A blueprint answers every question an engineering team needs answered before they write the first line of code:

- What are we building and why?
- Who is it for, and at what scale?
- How is it structured architecturally?
- What does the data model look like?
- What are the API contracts?
- What are the security controls?
- What infrastructure does it run on?
- How do we build it, phase by phase?
- How do we know when we're done?

A blueprint is the difference between starting a project with clarity versus starting with confusion. It is the Tesla pre-delivery checklist, the SpaceX launch commit criteria, the ITAR-controlled design specification — except for software systems.

**A blueprint is an executable artifact.** Any sufficiently capable AI agent or engineering team can take a completed blueprint and begin building immediately, with no additional context required.

---

## Design Principles

### 1. Complete
A blueprint contains everything needed to build the system. Every architectural decision is recorded. Every interface is defined. Every ambiguity is resolved. If something is unknown at blueprint time, it is explicitly flagged as a deferred decision — never silently omitted.

A complete blueprint means the receiving team never has to ask "what did they mean by this?" or make assumptions that could derail the build.

### 2. Portable
A blueprint is consumable by any intelligent system or team. It does not assume a particular IDE, a particular AI platform, or a particular project management tool. The schema is language-agnostic, platform-agnostic, and team-agnostic.

Portability is enforced through the Handoff Protocol — a standardized process that transforms a blueprint into the exact format any target system expects.

### 3. Structured
Every section of a blueprint has a defined schema. Machine-readable JSON powers automation; human-readable prose powers comprehension. Both coexist in every section. A CI/CD pipeline can parse the blueprint's infrastructure section to provision resources; a developer can read the same section as a narrative.

Structure is not bureaucracy — it is the prerequisite for automation.

### 4. Versioned
Every blueprint has a semantic version number and a changelog. When the system evolves, the blueprint evolves with it. Old versions are preserved in the Blueprint Registry. The version history tells the story of how the architecture changed and why.

Versioning means that when a blueprint is handed off to a downstream system, that system always knows exactly which version of the truth it received.

### 5. Composable
Blueprints can reference and extend other blueprints. A microservices platform blueprint can reference individual service blueprints. An AI system blueprint can reference a data platform blueprint as a dependency. This composability mirrors how real systems are built — from components.

A `parent_blueprint_id` field enables inheritance. A `dependencies` section captures inter-blueprint relationships. The Blueprint Registry resolves references at handoff time.

### 6. Executable
Blueprint sections map directly to implementation tasks. The `implementation.phases` section is not a wish list — it is a sequenced, dependency-resolved work breakdown structure. Each phase maps to GitHub issues, Linear tickets, or Jira stories through the Handoff Adapters.

An executable blueprint collapses the gap between design and doing.

---

## Blueprint Anatomy

Every System Blueprint contains the following sections. Each section has a defined schema (see `schema.md`) and a defined purpose.

### `meta`
Identity and provenance of the blueprint. Contains the UUID, name, version, creation timestamp, status, architect name, tags, and optional parent blueprint reference. The `meta` section is what the Blueprint Registry indexes.

### `problem`
The validated problem definition. Contains the problem statement, business context, constraints (technical, business, regulatory), assumptions, and measurable success criteria. This section is produced by the Strategist team and anchors every subsequent architectural decision.

### `system`
High-level system identity. Name, description, domain, type classification, user personas with their needs and usage patterns, and scale targets (users, requests per second, data volume, uptime requirements). This section defines the envelope within which all architecture decisions must fit.

### `architecture`
The core of the blueprint. Contains the architectural style choice (with rationale), the component catalog (every service, module, and boundary), the interface map (how components communicate), data flow diagrams (as structured objects), external dependencies, and Architecture Decision Records (ADRs). Every ADR records a decision, the alternatives considered, and the rationale.

### `data`
Complete data architecture. Data models (entities, relationships, constraints), data stores (type, technology, access patterns), data pipelines (sources, transformations, sinks), and data governance policies (ownership, retention, PII handling, access controls).

### `api`
All API contracts, the authentication scheme, and the versioning strategy. Each API contract specifies endpoints, request/response schemas, error codes, rate limits, and SLAs. This section is the source of truth for all interface contracts between services and with external consumers.

### `security`
The complete security posture. Threat model (assets, threats, attack vectors), security controls (prevention, detection, response), compliance requirements, and a risk register with likelihood/impact/mitigation for each risk. This section is produced by the Security team and reviewed against the threat model before blueprint approval.

### `infrastructure`
Everything needed to run the system. Cloud provider and region strategy, infrastructure topology (compute, network, storage), CI/CD pipeline definition, observability plan (metrics, logs, traces, alerts), and disaster recovery plan (RTO, RPO, runbook references).

### `implementation`
The sequenced build plan. Phases with objectives and deliverables, milestones with acceptance criteria, team structure with roles and responsibilities, tech stack with version constraints and rationale, and effort estimates with confidence levels.

### `quality`
The quality covenant. Test strategy (unit, integration, e2e, performance, security), non-functional requirement targets (latency, throughput, availability, error rate), and the Definition of Done — the specific, measurable criteria that must be met before any milestone is considered complete.

### `handoff`
Handoff configuration. Target systems (which platforms will receive this blueprint), export formats, and delivery method. This section drives the Handoff Protocol execution.

---

## Blueprint Lifecycle

```
DRAFT → REVIEW → APPROVED → ACTIVE → DEPRECATED
```

### DRAFT
The blueprint is being produced by the SuperArchitect OS. Teams are contributing their sections. Sections may be incomplete. Not yet suitable for implementation.

**Who acts**: Commander + all specialist teams
**Duration**: One OS workflow execution (typically a single session)
**Exit criteria**: All required sections complete, no `[TBD]` in critical fields

### REVIEW
The blueprint is complete and undergoing validation. The Commander performs a completeness check. The Security team reviews the security section. The Tech Lead reviews the architecture section. Automated schema validation runs.

**Who acts**: Commander (validation), human architect (optional review)
**Duration**: Minutes (automated) to hours (human review)
**Exit criteria**: All validation checks pass, all critical decisions resolved

### APPROVED
The blueprint has passed review and is authorized for implementation. This is the version that gets handed off to downstream systems. No further changes without a new version increment.

**Who acts**: Human architect or Commander (if autonomous mode)
**Duration**: Point-in-time event
**Exit criteria**: Explicit approval recorded in `meta.approved_by` and `meta.approved_at`

### ACTIVE
The blueprint is actively being implemented. Issues have been created, teams are building. Changes to the blueprint at this stage require a patch version increment and a change log entry.

**Who acts**: Implementation team
**Duration**: Duration of the build project
**Exit criteria**: All milestones complete, system in production

### DEPRECATED
The system is decommissioned or superseded by a new blueprint version. Preserved in the Blueprint Registry for historical reference. Not used for new work.

---

## Blueprint Registry

The Blueprint Registry is the catalog of all blueprints produced by the SuperArchitect OS. It enables discovery, reuse, and similarity-based recommendations for future builds.

### Storage
Blueprints are stored as structured files at:
```
/home/user/superarchitect/registry/blueprints/{id}/
  blueprint.json         # Machine-readable full blueprint
  blueprint.md           # Human-readable rendered version
  summary.md             # Executive summary (auto-generated)
  changelog.md           # Version history
  handoff/               # Handoff packages for each target system
```

### Indexing
The registry index at `/home/user/superarchitect/registry/index.json` tracks:
- Blueprint ID, name, version, status
- Domain and system type tags
- Creation date and architect
- Parent blueprint references
- Similarity vectors (for the Intelligence layer)

### Discovery
Blueprints are discoverable by:
- **Full-text search** across name, description, and tags
- **Domain filter** (fintech, healthtech, developer tools, etc.)
- **System type filter** (API, web app, data platform, etc.)
- **Scale filter** (MVP, growth, enterprise)
- **Similarity search** via the Intelligence layer's Similarity Engine

---

## Handoff Protocol

The Handoff Protocol is the standardized process for transmitting a completed blueprint to a downstream system. It is deterministic, auditable, and reversible.

### Step 1: Blueprint Validation (Completeness Check)
Before any handoff, the Commander runs the validation suite against the blueprint:

```
Validation Checks:
  ✓ meta.id is a valid UUID
  ✓ meta.status is "approved"
  ✓ problem.statement is non-empty
  ✓ problem.success_criteria has >= 1 item
  ✓ system.users has >= 1 persona
  ✓ architecture.components has >= 1 component
  ✓ architecture.adrs has >= 1 ADR
  ✓ security.threat_model is non-empty
  ✓ implementation.phases has >= 1 phase
  ✓ quality.definition_of_done has >= 1 item
  ✓ No [TBD] or [PLACEHOLDER] in required fields
```

If validation fails, the Commander returns specific failures and will not proceed until they are resolved.

### Step 2: Target System Identification
The Commander identifies which systems will receive the blueprint based on `handoff.target_systems`. Each entry specifies a system type, an endpoint (if applicable), and configuration overrides.

Supported target systems and their identifiers:
- `claude-code` — Claude Code project activation
- `github` — GitHub repository + project setup
- `linear` — Linear project + epics + stories
- `jira` — Jira project + epic/story structure
- `langchain` — LangChain/LangGraph agent graph
- `autogen` — AutoGen multi-agent setup
- `crewai` — CrewAI crew definition
- `generic-api` — Any REST endpoint accepting blueprint JSON

### Step 3: Adapter Selection
Each target system has a dedicated Handoff Adapter in `/home/user/superarchitect/os/handoff/adapters/`. The Commander selects the appropriate adapter based on the target system identifier.

### Step 4: Transformation
The adapter transforms the blueprint into the target system's native format. This is a pure transformation — the blueprint is not modified. The transformation produces a Handoff Package specific to the target.

### Step 5: Delivery
The Handoff Package is delivered to the target system via the method specified in `handoff.delivery_method`:
- `file` — Written to disk at a specified path
- `api` — Posted to a REST endpoint
- `clipboard` — Copied to system clipboard (for manual use)
- `git-commit` — Committed to a repository

### Step 6: Acknowledgement
The target system (or the operator) confirms receipt. For automated targets (GitHub, Linear, Jira), the adapter captures the response (issue IDs, project URLs, etc.) and records them in `handoff/acknowledgement.json`. For manual targets (Claude Code, clipboard), the operator confirms.

---

## Example Blueprint Summary

**Blueprint: Multi-tenant SaaS Analytics Platform**
**Version**: 1.0.0 | **Status**: Approved | **Domain**: B2B SaaS / Analytics

**Problem**: Mid-market companies lack affordable, real-time analytics infrastructure. They need a multi-tenant platform that ingests event streams, stores time-series data, and serves dashboards with sub-second query latency — at a price point 10x below Snowflake.

**System**: Web application + API + data platform hybrid. 500 initial tenants, growing to 10,000. 1M events/day per tenant at peak. 99.9% uptime SLA.

**Architecture**: Event-driven microservices. 8 core services: ingestion-service, routing-service, storage-service, query-service, dashboard-service, tenant-service, auth-service, billing-service. Apache Kafka for event streaming. ClickHouse for time-series storage. React + Recharts for dashboards.

**Key ADRs**:
- ADR-001: ClickHouse over TimescaleDB for query performance at scale
- ADR-002: Tenant isolation via schema-per-tenant (not row-level security) for query performance
- ADR-003: Kafka over SQS for replay capability and consumer group flexibility

**Security**: SOC 2 Type II target. Tenant data isolation enforced at database, API, and network layers. Encryption at rest (AES-256) and in transit (TLS 1.3). RBAC with tenant-scoped permissions.

**Implementation**: 4 phases over 16 weeks. Phase 1: Core ingestion + storage (4 weeks). Phase 2: Query engine + basic dashboards (4 weeks). Phase 3: Multi-tenancy + auth + billing (4 weeks). Phase 4: Production hardening + observability (4 weeks).

**Handoff**: Claude Code (primary), GitHub (project setup), Linear (sprint planning).

---

## Integration with Commander

The Commander agent is responsible for producing, validating, and handing off blueprints. The integration works as follows:

1. **Blueprint initialization**: At workflow start, Commander creates a blank blueprint with `meta.status = "draft"` and a unique UUID.

2. **Section delegation**: As each specialist team completes its work, Commander extracts the structured output and populates the corresponding blueprint section.

3. **Progressive completion tracking**: Commander maintains a completion checklist for the blueprint. It knows at any point which sections are complete, partial, or not started.

4. **Validation gate**: Before transitioning the blueprint to `"approved"`, Commander runs the full validation suite. If any check fails, Commander re-engages the responsible team to resolve the gap.

5. **Blueprint finalization**: Once approved, Commander generates the human-readable markdown render and stores both in the Blueprint Registry.

6. **Handoff execution**: Commander executes the Handoff Protocol for each configured target system, capturing acknowledgements.

7. **Retrospective hook**: After handoff, Commander appends a build summary to the Intelligence layer's feedback log for pattern learning.

The blueprint is not a side effect of the Commander's work — it is the primary output. Every other artifact (diagrams, ADRs, specs) is subordinate to and contained within the blueprint.
