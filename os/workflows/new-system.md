# New System Build Workflow

**Version:** 1.0
**Trigger:** `/build-system <description>`
**Owner:** Commander Agent
**Last Updated:** 2026-03-29

---

## Purpose

This workflow orchestrates the end-to-end construction of a new system from a natural language description. It integrates with the SuperArchitect OS kernel pipeline and dispatches specialist teams at each phase.

---

## Prerequisites

- System description provided by the human operator
- CLAUDE.md loaded and OS activated
- Commander Agent instantiated
- Team registry validated via `os/manifest.md`

---

## Workflow Phases

### Phase 1: Discovery and Intent Extraction

**Owner:** Commander + Product Agent
**Duration:** Fast (minutes)
**Dependencies:** None

**Steps:**
1. Commander parses the system description
2. Commander classifies the system type:
   - SaaS Platform
   - API Service
   - Data Pipeline
   - AI/ML Platform
   - Enterprise Application
   - Developer Tool
   - Mobile Application
3. Commander activates Product Agent for requirements engineering
4. Product Agent produces:
   - Problem statement
   - Target user personas
   - Core functional requirements
   - Non-functional requirements (scale, performance, compliance)
   - Success metrics
   - MoSCoW prioritization of features
5. Commander validates requirements completeness

**Quality Gate:**
- [ ] Problem statement is clear and specific
- [ ] Target users are defined with real pain points
- [ ] At least 3 Must-Have requirements defined
- [ ] Non-functional requirements have numeric targets
- [ ] Success metrics are measurable

**Artifacts:**
- `docs/product/requirements.md` — Product Requirements Document
- `docs/product/personas.md` — User personas
- `docs/product/metrics.md` — Success metrics definition

---

### Phase 2: Architecture Design

**Owner:** Architect Agent
**Duration:** Medium (minutes)
**Dependencies:** Phase 1 complete

**Steps:**
1. Commander dispatches requirements to Architect Agent
2. Architect Agent loads `os/teams/architect/patterns.md` for pattern selection
3. Architect Agent determines:
   - System architecture pattern (monolith, microservices, event-driven, hybrid)
   - Service decomposition (bounded contexts)
   - Technology stack selection (language, framework, database, broker, cache)
   - API design (REST, gRPC, GraphQL, AsyncAPI)
   - Data model design
   - Infrastructure topology
4. Architect Agent produces Architecture Decision Records (ADRs) for key choices
5. Architect Agent produces system blueprint
6. Commander reviews blueprint against requirements

**Quality Gate:**
- [ ] Architecture addresses all Must-Have requirements
- [ ] Non-functional requirements achievable with chosen architecture
- [ ] ADRs document trade-offs for every significant decision
- [ ] Data model supports all identified use cases
- [ ] Security architecture defined (auth, authz, encryption)
- [ ] Scalability strategy defined with growth projections

**Artifacts:**
- `docs/architecture/blueprint.md` — System architecture blueprint
- `docs/architecture/adrs/` — Architecture Decision Records
- `docs/architecture/data-model.md` — Data model design
- `docs/architecture/api-contracts/` — API specifications

---

### Phase 3: Security Architecture and Threat Model

**Owner:** Security Agent
**Duration:** Short (minutes)
**Dependencies:** Phase 2 complete

**Steps:**
1. Commander dispatches architecture to Security Agent
2. Security Agent performs STRIDE threat modeling
3. Security Agent defines:
   - Authentication architecture
   - Authorization model (RBAC/ABAC)
   - Data classification and encryption strategy
   - Network security design
   - Secrets management approach
   - Compliance requirements mapping
4. Security Agent reviews architecture for vulnerabilities
5. Security Agent produces security requirements for Engineering

**Quality Gate:**
- [ ] Threat model covers all system components
- [ ] Authentication mechanism specified
- [ ] Authorization model defined with roles and permissions
- [ ] Data classification complete for all data stores
- [ ] Encryption at rest and in transit specified
- [ ] Compliance requirements identified and mapped to controls

**Artifacts:**
- `docs/security/threat-model.md` — STRIDE threat model
- `docs/security/auth-design.md` — Authentication and authorization design
- `docs/security/compliance-matrix.md` — Compliance controls mapping

---

### Phase 4: Implementation

**Owner:** Engineer Agent + Designer Agent
**Duration:** Long (the bulk of the work)
**Dependencies:** Phase 2 and Phase 3 complete

**Parallel Tracks:**

**Track A: Backend Implementation (Engineer Agent)**
1. Set up project structure and build system
2. Implement domain model
3. Implement API endpoints
4. Implement data access layer
5. Implement authentication and authorization
6. Implement background jobs and event handlers
7. Implement observability (logging, metrics, tracing)
8. Write unit and integration tests

**Track B: Frontend Implementation (Designer Agent + Engineer Agent)**
1. Set up frontend project and design system
2. Implement core layout and navigation
3. Implement feature screens
4. Implement API integration
5. Implement state management
6. Implement responsive design and accessibility
7. Write component tests

**Track C: Infrastructure (DevOps Agent)**
1. Define infrastructure as code
2. Set up CI/CD pipeline
3. Configure monitoring and alerting
4. Set up container orchestration
5. Configure secrets management
6. Set up environments (dev, staging, production)

**Quality Gate:**
- [ ] All unit tests pass
- [ ] All integration tests pass
- [ ] Code coverage meets thresholds (90% business logic)
- [ ] Linting passes with zero warnings
- [ ] Security scan clean (no critical/high findings)
- [ ] API contracts match specification
- [ ] Infrastructure deploys successfully

**Artifacts:**
- Source code (all services)
- Test suites (unit, integration, contract)
- Infrastructure as Code
- CI/CD pipeline configuration
- Docker/container configurations

---

### Phase 5: Quality Assurance

**Owner:** QA Agent
**Duration:** Medium
**Dependencies:** Phase 4 complete (at least one track)

**Steps:**
1. QA Agent loads `os/teams/qa/checklists.md`
2. QA Agent executes pre-release checklist
3. QA Agent runs:
   - Full unit test suite
   - Integration test suite
   - API contract tests
   - Security tests (OWASP Top 10)
   - Performance tests (load, stress)
   - Accessibility audit (WCAG 2.1 AA)
4. QA Agent documents findings
5. QA Agent reports defects to Commander for triage
6. Commander dispatches fixes to Engineering

**Quality Gate:**
- [ ] All tests pass
- [ ] Coverage thresholds met
- [ ] No critical or high severity defects
- [ ] Performance within defined budgets
- [ ] Security scan clean
- [ ] Accessibility audit passes

**Artifacts:**
- `docs/qa/test-report.md` — Test execution report
- `docs/qa/performance-report.md` — Performance test results
- `docs/qa/security-report.md` — Security scan results

---

### Phase 6: Documentation

**Owner:** All Agents (coordinated by Commander)
**Duration:** Short
**Dependencies:** Phase 4 and Phase 5 complete

**Steps:**
1. Engineer Agent writes API documentation
2. Architect Agent writes architecture overview
3. DevOps Agent writes deployment and operations runbook
4. Product Agent writes user-facing documentation
5. Commander synthesizes README and getting-started guide

**Quality Gate:**
- [ ] README explains what the system does and how to run it
- [ ] API documentation covers all endpoints
- [ ] Architecture documentation matches implementation
- [ ] Deployment guide is complete and tested
- [ ] Runbook covers common operational scenarios

**Artifacts:**
- `README.md` — Project overview and quick start
- `docs/api/` — API reference documentation
- `docs/architecture/` — Architecture documentation
- `docs/operations/runbook.md` — Operations runbook
- `docs/development/setup.md` — Developer setup guide

---

### Phase 7: Delivery and Handoff

**Owner:** Commander
**Duration:** Short
**Dependencies:** All previous phases complete

**Steps:**
1. Commander runs final quality validation across all artifacts
2. Commander verifies all quality gates passed
3. Commander produces delivery summary:
   - System description and capabilities
   - Architecture overview with diagrams
   - Technology stack summary
   - Setup and deployment instructions
   - Known limitations and future recommendations
   - Security posture summary
   - Performance characteristics
4. Commander presents delivery to human operator

**Quality Gate:**
- [ ] All 6 previous phase quality gates passed
- [ ] System builds and runs successfully
- [ ] Tests pass end-to-end
- [ ] Documentation is complete
- [ ] Security audit passed
- [ ] No unresolved critical issues

**Final Artifact:**
- Complete, deployable system with full documentation

---

## Workflow Configuration

### Parallelism
```
Phase 1 (Discovery)
  │
  ▼
Phase 2 (Architecture)
  │
  ▼
Phase 3 (Security) ──────────────────┐
  │                                   │
  ▼                                   ▼
Phase 4 Track A (Backend) ─────┐    Phase 4 Track C (Infrastructure)
Phase 4 Track B (Frontend) ────┤
                               │
                               ▼
                         Phase 5 (QA)
                               │
                               ▼
                         Phase 6 (Docs)
                               │
                               ▼
                         Phase 7 (Delivery)
```

### Escalation Points
- Phase 1: Business trade-offs requiring human input
- Phase 2: Build vs. buy decisions with significant cost implications
- Phase 4: Scope questions that change requirements
- Phase 7: Final delivery acceptance

### Timeout Policy
- If any phase exceeds 3x its estimated duration, Commander escalates
- If a quality gate fails 3 times, Commander escalates with analysis

---

*This workflow builds systems end-to-end with zero hand-holding. Every phase has clear ownership, quality gates, and artifacts. The Commander orchestrates; the teams execute.*
