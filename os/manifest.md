# SuperArchitect OS — Configuration Manifest

> *The machine-readable registry of all OS components. Loaded by the Commander during boot.*

---

## OS Metadata

```yaml
os:
  name: SuperArchitect OS
  version: 1.0.0
  build_date: 2026-03-28
  schema_version: 1
  compatibility:
    min_context_window: 100000   # tokens — minimum for reliable OS operation
    recommended_context: 200000  # tokens — recommended for full parallel team operation
    native_runtime: claude-code
    supported_runtimes:
      - claude-code       # native, no adapter required
      - openai-codex      # adapter: os/adapters/openai-codex.md
      - gemini            # adapter: os/adapters/gemini.md
      - local-llm         # adapter: os/adapters/local-llm.md
  autonomy_default: FULL
  escalation_enabled: true
  audit_log_enabled: true
```

---

## Team Registry

All 10 specialist teams are registered below. The Commander consults this registry when routing tasks.

### Team 01 — Foundation

```yaml
team_id: team-01-foundation
name: Foundation
persona_file: os/teams/team-1-foundation/FOUNDATION.md
status: active
version: 1.0.0

capabilities:
  - os-maintenance
  - directory-structure
  - manifest-management
  - documentation-architecture
  - os-extension
  - versioning

trigger_conditions:
  - OS extension requested
  - New team or workflow to be added
  - Root-level documentation needs updating
  - OS versioning or migration required

input_schema:
  - extension_request: description of what to add to the OS
  - current_manifest: current os/manifest.md contents

output_schema:
  - new_or_updated_files: list of created/modified files
  - manifest_diff: changes to os/manifest.md
  - summary: human-readable change summary

dependencies_before: []
dependencies_after: []
max_parallel_instances: 1
```

### Team 02 — Architecture

```yaml
team_id: team-02-architecture
name: Architecture
persona_file: os/teams/team-2-architecture/ARCHITECTURE.md
status: active
version: 1.0.0

capabilities:
  - system-design
  - service-decomposition
  - data-modeling
  - api-contract-design
  - integration-pattern-selection
  - scalability-planning
  - adr-authoring
  - technology-selection

trigger_conditions:
  - New system build initiated
  - Significant feature requiring architectural change
  - System audit requires architecture review
  - Service decomposition decision needed
  - Technology trade-off resolution required

input_schema:
  - product_brief: product description and requirements
  - research_report: optional technology research from Team 10
  - constraints: budget, timeline, team size, existing tech stack
  - non_functional_requirements: scale, availability, latency targets

output_schema:
  - architecture_blueprint: complete system design document
  - service_catalog: list of services with responsibilities and contracts
  - data_model: entity definitions and relationships
  - api_contracts: service interface definitions
  - adr_log: architecture decisions made with rationale
  - tech_stack: selected technologies with justification

dependencies_before:
  - team-09-product (product brief)
  - team-10-research (technology research, optional)
dependencies_after:
  - team-03-backend
  - team-04-frontend
  - team-05-data
  - team-06-devops
  - team-07-security
max_parallel_instances: 1
```

### Team 03 — Backend

```yaml
team_id: team-03-backend
name: Backend
persona_file: os/teams/team-3-backend/BACKEND.md
status: active
version: 1.0.0

capabilities:
  - api-implementation
  - business-logic
  - service-communication
  - data-access-layer
  - background-jobs
  - caching-strategy
  - event-driven-architecture
  - runtime-configuration

trigger_conditions:
  - Backend services need implementation
  - API design is complete and approved
  - Service-to-service communication patterns needed
  - Background processing requirements identified

input_schema:
  - architecture_blueprint: from Team 2
  - api_contracts: from Team 2
  - data_model: from Team 2
  - tech_stack: selected backend technologies

output_schema:
  - service_implementations: complete service code with tests
  - api_specifications: detailed API docs (OpenAPI/AsyncAPI)
  - data_access_layer: ORM models, query patterns, migration scripts
  - configuration_schema: all env vars and config parameters
  - background_job_specs: job definitions and schedules

dependencies_before:
  - team-02-architecture
dependencies_after:
  - team-08-qa
  - team-06-devops
max_parallel_instances: 3
```

### Team 04 — Frontend

```yaml
team_id: team-04-frontend
name: Frontend
persona_file: os/teams/team-4-frontend/FRONTEND.md
status: active
version: 1.0.0

capabilities:
  - web-application-development
  - mobile-application-development
  - design-system-creation
  - component-library
  - state-management
  - performance-optimization
  - accessibility-compliance
  - progressive-web-apps

trigger_conditions:
  - User-facing interface required
  - Design system needs to be built or extended
  - Web or mobile application implementation needed
  - Frontend performance optimization required

input_schema:
  - architecture_blueprint: from Team 2
  - api_contracts: from Team 2 (backend contracts)
  - product_designs: wireframes, mockups from Team 9
  - design_tokens: colors, typography, spacing from Team 9

output_schema:
  - application_code: complete frontend application
  - component_library: reusable component documentation
  - state_management_spec: store definitions and data flows
  - performance_baseline: Core Web Vitals targets and measurements
  - accessibility_report: WCAG 2.1 AA compliance verification

dependencies_before:
  - team-02-architecture
  - team-09-product
dependencies_after:
  - team-08-qa
max_parallel_instances: 2
```

### Team 05 — Data

```yaml
team_id: team-05-data
name: Data
persona_file: os/teams/team-5-data/DATA.md
status: active
version: 1.0.0

capabilities:
  - data-pipeline-design
  - etl-elt-implementation
  - data-warehouse-design
  - ml-model-development
  - feature-store-management
  - analytics-infrastructure
  - data-quality-framework
  - data-lineage

trigger_conditions:
  - Data pipelines required
  - Analytics or reporting infrastructure needed
  - ML/AI features identified in product brief
  - Data warehouse or lakehouse design needed
  - Data quality issues identified in audit

input_schema:
  - architecture_blueprint: from Team 2
  - data_model: from Team 2
  - data_sources: descriptions of source systems
  - ml_requirements: model types, accuracy targets, latency requirements

output_schema:
  - pipeline_specs: complete ETL/ELT pipeline definitions
  - schema_definitions: data warehouse/lake schemas
  - ml_model_cards: model documentation and performance metrics
  - data_quality_rules: validation rules and thresholds
  - data_catalog: dataset documentation and lineage

dependencies_before:
  - team-02-architecture
dependencies_after:
  - team-06-devops
  - team-08-qa
max_parallel_instances: 2
```

### Team 06 — DevOps

```yaml
team_id: team-06-devops
name: DevOps
persona_file: os/teams/team-6-devops/DEVOPS.md
status: active
version: 1.0.0

capabilities:
  - ci-cd-pipeline-design
  - containerization
  - kubernetes-orchestration
  - cloud-infrastructure
  - infrastructure-as-code
  - monitoring-and-alerting
  - secret-management
  - disaster-recovery
  - auto-scaling

trigger_conditions:
  - Infrastructure specification needed
  - CI/CD pipeline setup required
  - Deployment strategy definition needed
  - Monitoring and observability stack required
  - Disaster recovery planning needed

input_schema:
  - architecture_blueprint: from Team 2
  - service_catalog: from Team 2
  - tech_stack: selected infrastructure technologies
  - sla_requirements: availability and RTO/RPO targets

output_schema:
  - infrastructure_code: Terraform/Pulumi/CDK definitions
  - ci_cd_pipelines: pipeline definitions (GitHub Actions, etc.)
  - kubernetes_manifests: deployment, service, ingress specs
  - monitoring_stack: dashboards, alerts, runbooks
  - secret_management_spec: vault/secret manager configuration
  - disaster_recovery_plan: backup, restore, and failover procedures

dependencies_before:
  - team-02-architecture
  - team-03-backend (partial — service specs needed)
dependencies_after:
  - commander (final deployment)
max_parallel_instances: 2
```

### Team 07 — Security

```yaml
team_id: team-07-security
name: Security
persona_file: os/teams/team-7-security/SECURITY.md
status: active
version: 1.0.0

capabilities:
  - threat-modeling
  - authentication-design
  - authorization-framework
  - encryption-implementation
  - dependency-auditing
  - penetration-testing
  - compliance-mapping
  - security-monitoring
  - certificate-management

trigger_conditions:
  - Any new system build (always activated)
  - Security audit requested
  - Compliance requirements identified (SOC2, GDPR, HIPAA, PCI-DSS)
  - Authentication or authorization design needed
  - Vulnerability detected in dependency scan

input_schema:
  - architecture_blueprint: from Team 2
  - service_catalog: from Team 2
  - compliance_requirements: list of applicable frameworks
  - threat_surface: user types, data classifications, external integrations

output_schema:
  - threat_model: STRIDE analysis per service
  - auth_spec: authentication and authorization design
  - security_controls: implemented and recommended controls
  - compliance_report: coverage mapping against required frameworks
  - vulnerability_report: dependency scan results and mitigations
  - security_runbook: incident response procedures

dependencies_before:
  - team-02-architecture
dependencies_after:
  - commander (security sign-off before deployment)
max_parallel_instances: 1
```

### Team 08 — QA

```yaml
team_id: team-08-qa
name: QA
persona_file: os/teams/team-8-qa/QA.md
status: active
version: 1.0.0

capabilities:
  - test-strategy-design
  - unit-test-authoring
  - integration-test-authoring
  - end-to-end-test-authoring
  - performance-testing
  - load-testing
  - chaos-engineering
  - visual-regression-testing
  - test-infrastructure

trigger_conditions:
  - Implementation phase begins (always activated alongside teams 3-5)
  - Test coverage gate not met
  - Regression introduced by change
  - Performance degradation detected
  - QA audit requested

input_schema:
  - service_implementations: from Team 3
  - api_specifications: from Team 3
  - frontend_application: from Team 4
  - user_stories: from Team 9
  - coverage_requirements: from os/standards/testing-standards.md

output_schema:
  - test_plan: complete test strategy document
  - test_suites: all test code organized by type
  - coverage_report: coverage metrics per service
  - performance_baseline: load test results and capacity model
  - test_infrastructure: test environment and fixture management code
  - qa_sign_off: formal approval with conditions noted

dependencies_before:
  - team-03-backend (for backend tests)
  - team-04-frontend (for frontend tests)
dependencies_after:
  - commander (QA sign-off required before deployment)
max_parallel_instances: 2
```

### Team 09 — Product

```yaml
team_id: team-09-product
name: Product
persona_file: os/teams/team-9-product/PRODUCT.md
status: active
version: 1.0.0

capabilities:
  - product-brief-authoring
  - user-story-mapping
  - user-flow-design
  - wireframing
  - information-architecture
  - design-token-specification
  - product-roadmap-planning
  - success-metrics-definition

trigger_conditions:
  - New system build initiated (always activated first)
  - Product brief ambiguous or incomplete
  - User experience design needed
  - Product roadmap planning required

input_schema:
  - build_request: original system description from human
  - research_report: market and competitive research from Team 10

output_schema:
  - product_brief: complete product specification
  - user_stories: prioritized user story backlog
  - user_flows: key interaction flows
  - wireframes: structural interface designs
  - design_tokens: visual design system foundations
  - success_metrics: KPIs and measurement plan

dependencies_before:
  - team-10-research (optional, can run in parallel)
dependencies_after:
  - team-02-architecture
  - team-04-frontend
max_parallel_instances: 1
```

### Team 10 — Research

```yaml
team_id: team-10-research
name: Research
persona_file: os/teams/team-10-research/RESEARCH.md
status: active
version: 1.0.0

capabilities:
  - technology-evaluation
  - build-vs-buy-analysis
  - competitive-analysis
  - architecture-pattern-research
  - emerging-technology-assessment
  - knowledge-synthesis
  - tech-radar-maintenance

trigger_conditions:
  - Technology selection decision needed
  - Build-vs-buy decision required
  - Unfamiliar domain or technology encountered
  - Tech radar update needed
  - Competitive landscape analysis requested

input_schema:
  - research_question: specific question or decision to be researched
  - domain: system domain and business context
  - constraints: budget, timeline, team expertise

output_schema:
  - research_report: structured findings with recommendation
  - technology_comparison: side-by-side evaluation matrix
  - recommendation: clear verdict with reasoning
  - knowledge_base_update: new entries for os/knowledge/

dependencies_before: []
dependencies_after:
  - team-02-architecture
  - team-09-product
max_parallel_instances: 2
```

---

## Kernel Registry

### Phases
| Phase | File | Agents | OS Teams |
|-------|------|--------|----------|
| 01: Intent Compilation | os/kernel/phases/01_intent_compilation.md | Intent Analyst, Controller | Product, Researcher |
| 02: Success Model | os/kernel/phases/02_success_model.md | Success Model Architect, Controller | Product, Researcher |
| 03: Architecture Search | os/kernel/phases/03_architecture_search.md | Search Architect, Controller | Architect |
| 04: Comparative Reasoning | os/kernel/phases/04_comparative_reasoning.md | Comparative Reasoner, Controller | Architect, Security, Data |
| 05: Structural Synthesis | os/kernel/phases/05_structural_synthesis.md | Synthesis Architect, Failure Mode, Optimization | Architect, Data, Security, DevOps, Designer |
| 06: Audit & Routing | os/kernel/phases/06_audit_and_routing.md | Audit Architect, Mutation Architect | QA, Security, Reviewer |
| 07: Packaging | os/kernel/phases/07_packaging.md | Packaging Architect, Controller | Commander, Designer |

### Schemas
- os/kernel/schemas/intent.schema.json
- os/kernel/schemas/success_model.schema.json
- os/kernel/schemas/candidate_architecture.schema.json
- os/kernel/schemas/audit_vector.schema.json
- os/kernel/schemas/evolution_ledger_entry.schema.json
- os/kernel/schemas/canonical_system_package.schema.json

### Runtime Modes
| Mode | Description | Candidate Count | Audit Threshold |
|------|-------------|-----------------|-----------------|
| Standard | Balanced exploration and correction | 3 | 0.75 |
| Search Heavy | Deep exploration, more candidates | 5+ | 0.75 |
| Conservative | Strict thresholds, more escalation | 3 | 0.85 |
| Overdrive | Focused specialist passes on weak dimensions | 3+ | 0.70 |

---

## Workflow Registry

```yaml
workflow_registry:

  - id: build-saas
    name: Build SaaS Product
    file: os/workflows/build-saas.md
    trigger: /build-system <saas product description>
    phases: 5
    teams_involved: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
    estimated_complexity: high
    typical_duration: full-context session

  - id: build-ai-platform
    name: Build AI/ML Platform
    file: os/workflows/build-ai-platform.md
    trigger: /build-system <ai platform description>
    phases: 6
    teams_involved: [2, 3, 4, 5, 6, 7, 8, 9, 10]
    estimated_complexity: very-high
    typical_duration: multi-session

  - id: build-enterprise
    name: Build Enterprise Microservices
    file: os/workflows/build-enterprise.md
    trigger: /build-system <enterprise system description>
    phases: 7
    teams_involved: [2, 3, 4, 5, 6, 7, 8, 9, 10]
    estimated_complexity: very-high
    typical_duration: multi-session

  - id: build-data-pipeline
    name: Build Data Pipeline
    file: os/workflows/build-data-pipeline.md
    trigger: /build-system <data pipeline description>
    phases: 4
    teams_involved: [2, 5, 6, 7, 8, 10]
    estimated_complexity: medium
    typical_duration: single-session

  - id: audit-system
    name: Audit Existing System
    file: os/workflows/audit-system.md
    trigger: /audit-system <system description or path>
    phases: 3
    teams_involved: [2, 7, 8, 10]
    estimated_complexity: medium
    typical_duration: single-session
```

---

## Standards Registry

```yaml
standards_registry:

  - id: code-quality
    name: Code Quality Standards
    file: os/standards/code-quality.md
    enforced_by: [team-03-backend, team-04-frontend, team-05-data]
    gate_level: REQUIRED

  - id: api-design
    name: API Design Standards
    file: os/standards/api-design.md
    enforced_by: [team-02-architecture, team-03-backend]
    gate_level: REQUIRED

  - id: security-baseline
    name: Security Baseline
    file: os/standards/security-baseline.md
    enforced_by: [team-07-security]
    gate_level: REQUIRED — blocks deployment

  - id: testing-standards
    name: Testing Standards
    file: os/standards/testing-standards.md
    enforced_by: [team-08-qa]
    gate_level: REQUIRED — blocks deployment

  - id: documentation
    name: Documentation Standards
    file: os/standards/documentation.md
    enforced_by: [all teams]
    gate_level: RECOMMENDED — warning if not met
```

---

## Adapter Registry

```yaml
adapter_registry:

  - id: openai-codex
    name: OpenAI Codex Adapter
    file: os/adapters/openai-codex.md
    runtime: openai-codex
    status: stable
    capability_coverage: 85%
    known_gaps: [parallel-dispatch, stateful-context]

  - id: gemini
    name: Google Gemini Adapter
    file: os/adapters/gemini.md
    runtime: gemini
    status: beta
    capability_coverage: 80%
    known_gaps: [file-system-ops, long-context-chunking]

  - id: local-llm
    name: Local LLM Adapter
    file: os/adapters/local-llm.md
    runtime: ollama, lm-studio
    status: experimental
    capability_coverage: 60%
    known_gaps: [complex-reasoning, large-context, parallel-dispatch]
```

---

## Global Constants and Configuration

```yaml
global_config:

  # Parallelism
  max_parallel_teams: 5           # Maximum teams active simultaneously
  max_parallel_tasks_per_team: 3  # Maximum parallel tasks within one team
  default_dispatch_mode: parallel # parallel | sequential

  # Autonomy
  default_autonomy_level: FULL    # FULL | SUPERVISED | MANUAL
  escalation_threshold: high      # low | medium | high | critical
  human_approval_required_for:
    - irreversible_destructive_operations
    - cost_exceeding_threshold
    - compliance_decisions
    - explicit_escalation_points

  # Context and Memory
  context_carry_forward: true     # Pass prior phase context to subsequent phases
  context_compression: true       # Compress context when approaching window limit
  max_context_per_task: 50000     # tokens — maximum context in a single TASK message
  checkpoint_interval: per_phase  # When to save build state checkpoints

  # Quality Gates
  min_test_coverage: 80           # Percent — minimum for deployment gate
  security_gate: required         # required | recommended | optional
  documentation_gate: recommended

  # Token Budgets (approximate, in tokens)
  budget_per_team_activation: 20000
  budget_per_workflow: 200000
  budget_per_full_build: 500000

  # Timeouts and Retries
  task_timeout_minutes: 30
  max_retries_per_task: 3
  retry_backoff: exponential      # none | linear | exponential

  # Output Format
  default_output_format: markdown
  artifact_versioning: timestamp-hash
  require_self_review: true

  # Logging
  audit_log_path: os/logs/
  log_level: INFO                 # DEBUG | INFO | WARN | ERROR
  log_decisions: true
  log_escalations: true
```

---

## Extension Points

The following extension points are defined for OS customization without modifying core files:

```yaml
extension_points:

  # Custom team definitions
  - id: custom-teams
    type: directory
    path: os/teams/custom/
    description: Place custom team definitions here. They will be auto-discovered
                 and available for Commander dispatch without modifying manifest.md.

  # Custom workflows
  - id: custom-workflows
    type: directory
    path: os/workflows/custom/
    description: Place custom workflow files here. Invoke with /run-workflow <filename>.

  # Custom patterns
  - id: custom-patterns
    type: directory
    path: os/patterns/custom/
    description: Project-specific patterns that don't belong in the shared library.

  # Project-level overrides
  - id: project-config
    type: file
    path: .superarchitect.yaml
    description: Place this file at the project root to override any global_config
                 value for this specific project. Commander checks for this file
                 during boot and merges it with os/manifest.md settings.

  # Build hooks
  - id: pre-phase-hook
    type: file_pattern
    path: os/hooks/pre-phase-*.md
    description: Files matching this pattern are loaded by Commander before each
                 phase begins. Use for project-specific setup or validation steps.

  - id: post-phase-hook
    type: file_pattern
    path: os/hooks/post-phase-*.md
    description: Loaded by Commander after each phase completes. Use for
                 cross-cutting concerns like sending notifications or updating trackers.
```

---

*SuperArchitect OS v1.0 — os/manifest.md — Foundation Team (Team 1) — 2026-03-28*
