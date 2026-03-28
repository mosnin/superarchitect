# SuperArchitect OS — Directory Guide

> *A complete reference for every file in the OS, how teams collaborate, and how to extend the system.*

---

## Purpose of This Document

This guide is the definitive reference for navigating the SuperArchitect OS file system. It documents every directory, every file, and the intent behind each. Any agent or human engineer working with this OS should read this document before exploring the directory tree.

---

## Full Annotated Directory Tree

```
superarchitect/
│
├── CLAUDE.md
│   The root OS manifest. The single most important file. Reading this file
│   activates the OS and transforms the AI agent into the SuperArchitect
│   Commander. Contains: OS identity, core principles, agent hierarchy,
│   architecture overview, boot sequence, quick start commands, and the
│   complete operating instructions.
│
└── os/
    │
    ├── README.md  (this file)
    │   The directory guide. Annotates every file and directory.
    │   Reference for orientation, collaboration patterns, and extension.
    │
    ├── manifest.md
    │   The OS configuration manifest. Machine-readable registry of all teams,
    │   workflows, standards, adapters, and global constants. The Commander
    │   loads this file during boot to initialize its internal registry.
    │
    ├── commander/
    │   The Commander agent's operational files. This directory contains
    │   everything needed to fully instantiate and operate the Commander.
    │   │
    │   ├── COMMANDER.md
    │   │   The Commander's persona, responsibilities, decision authority,
    │   │   dispatch rules, synthesis protocol, quality gates, escalation
    │   │   protocol, and communication format. This is the Commander's
    │   │   primary instruction set.
    │   │
    │   ├── dispatch.md
    │   │   Task classification taxonomy, routing decision trees, parallel
    │   │   vs. sequential dispatch rules, dependency graph management,
    │   │   load balancing heuristics, priority levels, and example
    │   │   dispatch scenarios for different system types.
    │   │
    │   └── protocols.md
    │       Inter-agent communication protocols: message schema, message
    │       types, handoff protocol, context passing format, error handling,
    │       retry/recovery protocol, agent state machine, and output
    │       versioning.
    │
    ├── teams/
    │   One subdirectory per specialist team. Each team directory contains
    │   the team's persona file, capability map, and standard operating
    │   procedures.
    │   │
    │   ├── team-1-foundation/
    │   │   ├── FOUNDATION.md       # Team persona and OS ownership responsibilities
    │   │   └── sop.md              # Standard operating procedures for OS maintenance
    │   │
    │   ├── team-2-architecture/
    │   │   ├── ARCHITECTURE.md     # Team persona: system design, ADRs, blueprints
    │   │   ├── templates/
    │   │   │   ├── adr-template.md         # Architecture Decision Record template
    │   │   │   ├── system-design.md        # System design document template
    │   │   │   └── api-contract.md         # API contract template
    │   │   └── patterns-index.md           # Index of architectural patterns this team uses
    │   │
    │   ├── team-3-backend/
    │   │   ├── BACKEND.md          # Team persona: APIs, services, data layers
    │   │   ├── templates/
    │   │   │   ├── service-spec.md         # Service specification template
    │   │   │   └── api-implementation.md   # API implementation checklist
    │   │   └── tech-stack-defaults.md      # Default technology choices per project type
    │   │
    │   ├── team-4-frontend/
    │   │   ├── FRONTEND.md         # Team persona: web, mobile, design systems
    │   │   ├── templates/
    │   │   │   ├── component-spec.md       # Component specification template
    │   │   │   └── ux-checklist.md         # UX quality checklist
    │   │   └── accessibility-baseline.md   # Accessibility requirements (WCAG 2.1 AA)
    │   │
    │   ├── team-5-data/
    │   │   ├── DATA.md             # Team persona: pipelines, ML, analytics
    │   │   ├── templates/
    │   │   │   ├── pipeline-spec.md        # Data pipeline specification template
    │   │   │   └── ml-model-card.md        # ML model documentation template
    │   │   └── data-quality-standards.md   # Data quality expectations
    │   │
    │   ├── team-6-devops/
    │   │   ├── DEVOPS.md           # Team persona: CI/CD, infra, observability
    │   │   ├── templates/
    │   │   │   ├── pipeline-spec.md        # CI/CD pipeline specification template
    │   │   │   └── infra-spec.md           # Infrastructure specification template
    │   │   └── platform-defaults.md        # Default platform and tooling choices
    │   │
    │   ├── team-7-security/
    │   │   ├── SECURITY.md         # Team persona: threat modeling, auth, compliance
    │   │   ├── templates/
    │   │   │   ├── threat-model.md         # Threat model template
    │   │   │   └── security-review.md      # Security review checklist
    │   │   └── compliance-matrix.md        # Compliance framework coverage matrix
    │   │
    │   ├── team-8-qa/
    │   │   ├── QA.md               # Team persona: test strategy, quality gates
    │   │   ├── templates/
    │   │   │   ├── test-plan.md            # Test plan template
    │   │   │   └── test-report.md          # Test results report template
    │   │   └── coverage-requirements.md    # Minimum coverage thresholds
    │   │
    │   ├── team-9-product/
    │   │   ├── PRODUCT.md          # Team persona: UX research, product design
    │   │   ├── templates/
    │   │   │   ├── user-story.md           # User story template
    │   │   │   └── product-brief.md        # Product brief template
    │   │   └── design-principles.md        # Product design principles
    │   │
    │   └── team-10-research/
    │       ├── RESEARCH.md         # Team persona: tech evaluation, knowledge synthesis
    │       ├── templates/
    │       │   └── research-report.md      # Research report template
    │       └── evaluation-criteria.md      # Technology evaluation criteria
    │
    ├── workflows/
    │   Executable multi-phase build and audit workflows. Each workflow
    │   specifies exactly which teams are activated, in what order, and
    │   with what dependencies.
    │   │
    │   ├── build-saas.md
    │   │   Full build workflow for SaaS products. Covers: product definition,
    │   │   system architecture, backend APIs, frontend UI, data layer,
    │   │   infrastructure, security hardening, testing, and deployment.
    │   │
    │   ├── build-ai-platform.md
    │   │   Build workflow for AI/ML platforms. Extends build-saas.md with
    │   │   additional phases for: model selection, training pipeline, inference
    │   │   infrastructure, feature stores, evaluation harnesses, and MLOps.
    │   │
    │   ├── build-enterprise.md
    │   │   Build workflow for enterprise microservices systems. Covers:
    │   │   domain decomposition, service mesh, API gateway, event streaming,
    │   │   multi-region deployment, compliance controls, and runbook creation.
    │   │
    │   ├── build-data-pipeline.md
    │   │   Build workflow for data engineering systems. Covers: data source
    │   │   analysis, pipeline architecture, transformation logic, quality
    │   │   checks, monitoring, and data catalog.
    │   │
    │   └── audit-system.md
    │       Audit workflow for existing systems. Covers: architecture review,
    │       code quality analysis, security audit, dependency audit, performance
    │       analysis, test coverage review, and remediation planning.
    │
    ├── standards/
    │   Non-negotiable quality and engineering standards. All teams reference
    │   these standards when producing outputs. They define the floor, not
    │   the ceiling.
    │   │
    │   ├── code-quality.md
    │   │   Naming conventions, complexity limits, comment standards, linter
    │   │   configurations, code review criteria, and refactoring triggers.
    │   │
    │   ├── api-design.md
    │   │   REST API design rules, GraphQL schema conventions, versioning
    │   │   strategy, error response format, pagination standards, rate
    │   │   limiting requirements, and OpenAPI documentation requirements.
    │   │
    │   ├── security-baseline.md
    │   │   Minimum security requirements: authentication, authorization,
    │   │   input validation, secret management, TLS requirements, audit
    │   │   logging, and vulnerability scanning.
    │   │
    │   ├── testing-standards.md
    │   │   Coverage requirements, test naming conventions, test isolation
    │   │   rules, mock usage guidelines, fixture management, test data
    │   │   strategy, and CI gate thresholds.
    │   │
    │   └── documentation.md
    │       README requirements, inline documentation standards, API
    │       reference requirements, ADR format, runbook structure, and
    │       onboarding guide checklist.
    │
    ├── patterns/
    │   A curated library of architectural and implementation patterns.
    │   Teams reference patterns rather than reinventing solutions.
    │   │
    │   ├── architectural/
    │   │   ├── event-sourcing.md
    │   │   ├── cqrs.md
    │   │   ├── saga.md
    │   │   ├── api-gateway.md
    │   │   ├── strangler-fig.md
    │   │   ├── sidecar.md
    │   │   └── bulkhead.md
    │   │
    │   ├── implementation/
    │   │   ├── repository.md
    │   │   ├── unit-of-work.md
    │   │   ├── circuit-breaker.md
    │   │   ├── outbox.md
    │   │   └── feature-flags.md
    │   │
    │   └── anti-patterns/
    │       ├── distributed-monolith.md
    │       ├── god-service.md
    │       ├── chatty-api.md
    │       ├── shared-database.md
    │       └── magic-strings.md
    │
    ├── knowledge/
    │   The OS knowledge base. Growing library of case studies, technology
    │   evaluations, and decision records that ground OS decisions in
    │   real-world evidence.
    │   │
    │   ├── case-studies/
    │   │   Analyzed real-world system architectures documenting design
    │   │   decisions, trade-offs, and lessons learned.
    │   │
    │   ├── tech-radar/
    │   │   Technology evaluations organized by category (languages, frameworks,
    │   │   databases, cloud services, tooling). Each entry includes: verdict
    │   │   (adopt/trial/assess/hold), rationale, and use-case fit.
    │   │
    │   └── decision-records/
    │       Templates and completed ADRs from past builds. Used by the
    │       Architecture team to avoid re-litigating settled decisions.
    │
    └── adapters/
        Runtime adapters for non-Claude environments. Each adapter translates
        the OS's native protocol into the instruction format of the target runtime.
        │
        ├── openai-codex.md
        │   Adapter for OpenAI Codex. Translates TASK messages into Codex
        │   prompt chains. Documents capability gaps and workarounds.
        │
        ├── gemini.md
        │   Adapter for Google Gemini. Handles context window management
        │   differences and tool-use translation.
        │
        └── local-llm.md
            Adapter for local LLMs (Ollama, LM Studio). Includes capability
            tiering guidance (which tasks require a capable model vs. which
            can run on smaller models).
```

---

## How Teams Collaborate

Teams in SuperArchitect OS do not operate in isolation. They are coordinated by the Commander and communicate through structured messages. Here is the canonical collaboration model:

### Phase-Based Collaboration

Most workflows divide work into phases. Teams within the same phase can work in parallel; teams in subsequent phases depend on outputs from prior phases.

**Example: SaaS Build (simplified)**
```
Phase 1 (parallel):   Team 9 (Product brief) + Team 10 (Tech research)
Phase 2 (sequential): Team 2 (Architecture — depends on Phase 1)
Phase 3 (parallel):   Team 3 (Backend) + Team 4 (Frontend) + Team 5 (Data)
                       — all depend on Phase 2 architecture blueprint
Phase 4 (parallel):   Team 6 (DevOps) + Team 7 (Security) + Team 8 (QA)
                       — can start with partial Phase 3 output
Phase 5 (sequential): Commander synthesis — depends on all Phase 4 complete
```

### Context Handoff

When Team A's output becomes Team B's input, the Commander packages Team A's results as a CONTEXT block and prepends it to Team B's TASK message. Teams never read each other's files directly — all context flows through the Commander.

### Conflict Resolution

When two teams produce conflicting outputs (e.g., Architecture specifies PostgreSQL, Backend proposes MongoDB), the Commander holds a synthesis session: reviewing both justifications, consulting the relevant pattern and standard files, and making a binding decision documented in the ADR log.

### Review Loops

Any team can request a QUERY to another team to resolve ambiguity. Queries are routed through the Commander. The queried team responds within its current activation context. Excessive queries (more than 3 per task) trigger an escalation — it signals insufficient context in the original task dispatch.

---

## How to Extend the OS

### Adding a New Team

1. Create directory: `os/teams/team-N-name/`
2. Create team persona file: `os/teams/team-N-name/TEAMNAME.md`
   - Follow the structure of existing team files
   - Define: identity, capabilities, trigger conditions, input schema, output schema, quality checklist
3. Create SOP file: `os/teams/team-N-name/sop.md`
4. Register the team in `os/manifest.md` under `team_registry`
5. Update `os/commander/dispatch.md` to include routing rules for the new team
6. Update `CLAUDE.md` Agent Hierarchy section

### Adding a New Workflow

1. Create workflow file: `os/workflows/workflow-name.md`
   - Define: trigger, input schema, phases, team assignments, quality gates, output schema
2. Register in `os/manifest.md` under `workflow_registry`
3. Update `CLAUDE.md` Quick Start table if it is a top-level command

### Adding a New Pattern

1. Determine the pattern category (architectural, implementation, or anti-pattern)
2. Create: `os/patterns/[category]/[pattern-name].md`
   - Include: intent, motivation, structure, participants, consequences, known uses, related patterns
3. Update the patterns index in the relevant team files that should reference this pattern

### Adding a New Standard

1. Create: `os/standards/[standard-name].md`
2. Update team files that must comply with this standard to reference it
3. Update the standards registry in `os/manifest.md`

### Adding a New Adapter

1. Create: `os/adapters/[runtime-name].md`
   - Document: capability map, message translation rules, context window limits, known gaps
2. Register in `os/manifest.md` under `adapter_registry`
3. Update `CLAUDE.md` Runtime Support section

---

## Versioning Philosophy

### OS Versioning

SuperArchitect OS uses semantic versioning: `MAJOR.MINOR.PATCH`

- **MAJOR**: Incompatible changes to the Commander protocol, message schema, or core architecture
- **MINOR**: New teams, workflows, patterns, or standards added in a backward-compatible way
- **PATCH**: Bug fixes, clarifications, documentation improvements, and minor corrections

The current version is recorded in `os/manifest.md`. All files carry the OS version at which they were created or last significantly modified.

### Artifact Versioning

Every artifact produced during a build (architecture blueprints, API contracts, infrastructure specs) is versioned using the timestamp and a short hash. This enables:
- Resuming interrupted builds from a known state
- Comparing outputs across multiple build runs
- Rolling back to a prior design if a new direction proves unworkable

### Team File Versioning

Team persona files and SOPs are versioned alongside the OS. When a team's capabilities expand, its file is updated and the minor version is incremented. Breaking changes to team interfaces increment the major version.

---

## Glossary

**Agent**: An AI model instance operating within a defined persona and instruction set. In SuperArchitect OS, the Commander and each specialist team are agents.

**Team**: A named agent persona with a defined capability domain, input/output schema, and operating procedures. Teams are instantiated on demand by the Commander.

**Workflow**: A structured, multi-phase execution plan that specifies which teams are activated, in what order, with what dependencies, to produce a complete system output.

**Protocol**: The formal communication rules governing message exchange between agents. All inter-agent communication follows the schema defined in `os/commander/protocols.md`.

**Adapter**: A translation layer that converts OS-native protocol messages into the instruction format of a specific AI runtime (e.g., OpenAI Codex, Gemini).

**Phase**: A discrete unit of work within a workflow that groups related team activations. Phases execute sequentially (one phase completes before the next begins). Tasks within a phase can execute in parallel.

**Artifact**: A structured document produced by a team as the output of a task. Examples: architecture blueprint, API contract, test plan, infrastructure specification.

**Quality Gate**: A defined condition that must be satisfied before the workflow advances to the next phase. Quality gates prevent defects from propagating forward.

**ADR (Architecture Decision Record)**: A structured document recording a significant architectural decision — its context, the decision made, and the consequences.

**Commander**: The top-level orchestrator agent. It receives all build requests, manages all team dispatches, resolves conflicts, enforces quality gates, and delivers final outputs.

**Context Block**: A structured summary of prior phase outputs that is prepended to a team's TASK message to give it the context it needs to execute correctly.

**Escalation**: A message from an agent to the human operator requesting a decision that falls outside the agent's autonomous authority.

**Dispatch**: The act of the Commander assigning a task to a specific team by sending it a TASK message with a fully specified work package.

**Handoff**: The act of one team's output becoming another team's input, mediated by the Commander through context packaging.

**Self-Review**: The mandatory quality pass an agent performs on its own output before declaring a task COMPLETE, measured against the applicable standards in `os/standards/`.

**Boot Sequence**: The initialization procedure executed when a new build is triggered. It loads manifests, validates registries, activates the Commander, and begins the first workflow phase.

---

*SuperArchitect OS v1.0 — os/README.md — Foundation Team (Team 1) — 2026-03-28*
