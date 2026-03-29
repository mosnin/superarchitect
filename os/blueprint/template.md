# System Blueprint Template

> Fill in every `[FIELD]` placeholder to produce a complete System Blueprint. Each section contains comments (marked with `<!-- -->`) explaining what belongs there. This template is designed to be completed by an AI agent or human architect with no additional context required.

---

# SYSTEM BLUEPRINT

**Blueprint ID**: `[AUTO-GENERATED UUID]`
**Name**: `[SYSTEM NAME — the proper name of the system being built]`
**Version**: `1.0.0`
**Created**: `[ISO 8601 TIMESTAMP]`
**Status**: `draft`
**Architect**: `[NAME OR "SuperArchitect OS v1.0"]`
**Tags**: `[COMMA-SEPARATED TAGS — domain, system type, key technologies, scale tier]`

---

## 1. Problem Definition

<!-- This section establishes WHY the system exists. It must be completed before any architecture work begins. Every architectural decision downstream should be traceable back to a constraint or success criterion defined here. -->

### Problem Statement

`[ONE TO THREE SENTENCES. State the problem in concrete, business-language terms. Who is suffering from this problem, what is the problem, and what is the cost of not solving it?]`

### Context

`[TWO TO FIVE PARAGRAPHS. Provide background: the market context, the current state of the world that creates this problem, any relevant history, and why now is the right time to solve it.]`

### Constraints

<!-- List every constraint that is non-negotiable. A constraint is a condition that must be satisfied — violating it is not an option. -->

| ID | Type | Title | Detail | Impact |
|----|------|-------|--------|--------|
| CON-001 | `[technical/business/regulatory/resource/time]` | `[SHORT TITLE]` | `[FULL DETAIL — what the constraint is and why it exists]` | `[blocking/high/medium/low]` |
| CON-002 | `[...]` | `[...]` | `[...]` | `[...]` |
| `[ADD MORE ROWS AS NEEDED]` | | | | |

### Assumptions

<!-- List every assumption the architecture makes. If an assumption is wrong, the architecture may need to change. Making assumptions explicit prevents silent failures. -->

| ID | Statement | Risk If Wrong | Owner |
|----|-----------|---------------|-------|
| ASM-001 | `[THE ASSUMPTION — state it as a fact that you believe to be true]` | `[WHAT BREAKS if this assumption is incorrect]` | `[WHO is responsible for validating this assumption]` |
| ASM-002 | `[...]` | `[...]` | `[...]` |

### Success Criteria

<!-- Every criterion must be measurable. "Fast" is not a success criterion. "p99 latency < 200ms at 10,000 concurrent users" is. -->

| ID | Title | Metric | Target | Measurement Method | Timeframe |
|----|-------|--------|--------|--------------------|-----------|
| SC-001 | `[CRITERION NAME]` | `[WHAT IS MEASURED]` | `[THE NUMERIC OR BOOLEAN TARGET]` | `[HOW IT IS MEASURED]` | `[WHEN IT MUST BE ACHIEVED]` |
| SC-002 | `[...]` | `[...]` | `[...]` | `[...]` | `[...]` |

---

## 2. System Definition

### System Overview

**Name**: `[SYSTEM NAME]`
**Description**: `[TWO TO FOUR SENTENCES describing what this system does, for whom, and the core value it delivers]`
**Domain**: `[PRIMARY BUSINESS DOMAIN — e.g., "Financial Services / Payment Processing"]`
**System Type**: `[web_app / api / data_platform / ai_system / cli / library / sdk / microservices_platform / event_pipeline / ml_platform / iot_backend / mobile_backend / hybrid]`

### User Personas

<!-- Define every distinct user type. "Users" are anyone who interacts with the system, including developers consuming an API, admins operating the system, and end customers using the product. -->

#### Persona: `[PERSONA NAME — e.g., "Data Analyst"]`
- **Role**: `[JOB TITLE OR FUNCTIONAL ROLE]`
- **Goals**:
  - `[PRIMARY GOAL]`
  - `[SECONDARY GOAL]`
  - `[...]`
- **Pain Points**:
  - `[PAIN POINT 1 — what frustrates them today]`
  - `[PAIN POINT 2]`
- **Technical Level**: `[non-technical / semi-technical / technical / expert]`
- **Usage Frequency**: `[real-time / daily / weekly / occasional]`
- **Critical Flows**:
  - `[FLOW 1 — the most important thing they do in the system]`
  - `[FLOW 2]`

#### Persona: `[PERSONA NAME]`
`[REPEAT STRUCTURE ABOVE]`

### Scale Targets

| Dimension | Initial | Peak | Notes |
|-----------|---------|------|-------|
| Active Users | `[NUMBER]` | `[NUMBER]` | `[CONTEXT]` |
| Requests/Second | `[NUMBER]` | `[NUMBER]` | `[CONTEXT]` |
| Data Volume (GB) | `[NUMBER]` | `[NUMBER/YEAR GROWTH]` | `[CONTEXT]` |
| Uptime Target | — | `[PERCENT, e.g., 99.9%]` | `[SLA obligation if any]` |
| Latency (p99) | — | `[MS]` | `[At what load level]` |
| Geographic Regions | `[LIST REGIONS]` | — | `[Expansion plans]` |

---

## 3. Architecture

### Architectural Style

**Style**: `[microservices / monolith / modular_monolith / event_driven / serverless / cqrs_event_sourcing / layered / hexagonal / space_based / service_mesh]`

**Rationale**: `[TWO TO FOUR SENTENCES explaining WHY this style was chosen over alternatives. What specific constraints or requirements drove this decision?]`

**ADR Reference**: `[ADR-001 — the formal ADR documenting this decision is below]`

### Component Catalog

<!-- Every discrete service, module, or infrastructure component that has its own deployment boundary, data ownership, or team ownership gets its own entry. -->

#### Component: `[COMPONENT NAME — e.g., "query-service"]`
- **ID**: `COMP-001`
- **Type**: `[service / gateway / worker / scheduler / cache / database / message_broker / cdn / load_balancer / frontend / cli / sdk / ml_model / data_pipeline]`
- **Responsibility**: `[TWO TO FOUR SENTENCES. What does this component do? What is it responsible for? What is explicitly NOT its responsibility (anti-responsibilities)?]`
- **Owns Data**: `[LIST OF DOMAIN ENTITIES this component owns — it is the source of truth for these]`
- **Exposes**: `[LIST OF INTERFACES/APIs this component offers to others]`
- **Consumes**: `[LIST OF INTERFACES/APIs this component calls]`
- **Tech Stack**: `[TECHNOLOGIES AND VERSIONS — e.g., "Go 1.22, Redis 7, Prometheus"]`
- **Scaling Strategy**: `[horizontal / vertical / auto / fixed]`
- **Criticality**: `[critical / high / medium / low]`
- **Team Owner**: `[TEAM NAME]`

#### Component: `[COMPONENT NAME]`
`[REPEAT STRUCTURE ABOVE FOR EACH COMPONENT]`

### Interface Map

<!-- An interface is the contract between two components. Every integration between components must be listed here. -->

| ID | Name | Type | Protocol | Version | Producer | Consumers | Contract Ref |
|----|------|------|----------|---------|----------|-----------|--------------|
| INT-001 | `[NAME]` | `[rest_api/grpc/graphql/event/websocket]` | `[HTTP/gRPC/AMQP/...]` | `[v1]` | `[COMP-ID]` | `[COMP-IDs]` | `[API-ID]` |

### Data Flows

<!-- Document the key flows of data through the system. Focus on the most important business operations — the paths that, if broken, would have the highest impact. -->

#### Flow: `[FLOW NAME — e.g., "Event Ingestion Flow"]`
- **Trigger**: `[WHAT INITIATES THIS FLOW]`
- **Steps**:
  1. `[COMPONENT A]` → `[COMPONENT B]` via `[INTERFACE]`: `[WHAT DATA IS SENT AND WHY]`
  2. `[COMPONENT B]` → `[COMPONENT C]` via `[INTERFACE]`: `[...]`
  3. `[...]`
- **Data Types**: `[LIST OF ENTITY TYPES INVOLVED]`
- **PII Involved**: `[YES/NO — if yes, describe the PII and how it's protected]`

### External Dependencies

| ID | Name | Type | Purpose | Version | Risk | Fallback |
|----|------|------|---------|---------|------|---------|
| DEP-001 | `[NAME — e.g., "Stripe"]` | `[saas/open_source/internal_service]` | `[WHY IT'S NEEDED]` | `[VERSION OR "latest"]` | `[critical/high/medium/low]` | `[WHAT HAPPENS IF IT'S UNAVAILABLE]` |

### Architecture Decision Records

#### ADR-001: `[DECISION TITLE — e.g., "Use ClickHouse for Time-Series Storage"]`
- **Date**: `[ISO 8601 DATE]`
- **Status**: `[proposed/accepted/superseded/deprecated]`
- **Context**: `[THE SITUATION that forced this decision. What is the problem? What are the constraints?]`
- **Decision**: `[THE CHOICE MADE. State it unambiguously.]`
- **Rationale**: `[WHY this option over alternatives. Reference the constraints and success criteria that drove the choice.]`
- **Alternatives Considered**:
  - `[ALTERNATIVE 1]`: Pros: `[...]` | Cons: `[...]` | Rejected because: `[...]`
  - `[ALTERNATIVE 2]`: Pros: `[...]` | Cons: `[...]` | Rejected because: `[...]`
- **Consequences**:
  - `[POSITIVE CONSEQUENCE]`
  - `[NEGATIVE CONSEQUENCE OR TRADE-OFF TO MANAGE]`

`[ADD ADR-002, ADR-003, ... FOR EACH SIGNIFICANT ARCHITECTURAL DECISION]`

---

## 4. Data Architecture

### Data Models

#### Entity: `[ENTITY NAME — e.g., "Tenant"]`

| Field | Type | Required | Unique | PII | Encrypted | Description |
|-------|------|----------|--------|-----|-----------|-------------|
| `id` | `UUID` | Yes | Yes | No | No | `Primary identifier` |
| `[FIELD NAME]` | `[TYPE]` | `[Y/N]` | `[Y/N]` | `[Y/N]` | `[Y/N]` | `[DESCRIPTION]` |

**Relationships**:
- `[ENTITY NAME]` has many `[RELATED ENTITY]` via `[FOREIGN KEY]`

`[ADD ENTITY TABLES FOR EACH CORE DOMAIN ENTITY]`

### Data Stores

| ID | Name | Type | Technology | Version | Purpose | Estimated Size | Backup |
|----|------|------|-----------|---------|---------|----------------|--------|
| DS-001 | `[NAME]` | `[relational/document/key_value/time_series/graph/vector/object/search]` | `[TECH — e.g., PostgreSQL]` | `[VERSION]` | `[WHY THIS STORE EXISTS]` | `[GB]` | `[FREQUENCY]` |

### Data Pipelines

| ID | Name | Type | Trigger | Source | Sink | Technology | Latency SLA |
|----|------|------|---------|--------|------|-----------|-------------|
| PIP-001 | `[NAME]` | `[batch/streaming/micro_batch]` | `[schedule/event/api/continuous]` | `[SOURCE STORE/SYSTEM]` | `[SINK STORE/SYSTEM]` | `[TECHNOLOGY]` | `[TARGET LATENCY]` |

### Data Governance

- **Data Owner**: `[TEAM OR ROLE responsible for data quality and policy]`
- **PII Fields**: `[LIST ALL FIELDS containing personally identifiable information]`
- **Retention Policies**:
  - `[DATA TYPE]`: Retain for `[DURATION]` — Reason: `[REGULATORY OR BUSINESS BASIS]`
- **Anonymization Strategy**: `[HOW PII IS ANONYMIZED for analytics/non-production use]`
- **Access Controls**:
  - `[ROLE]` can access `[DATA TYPE]` for `[PURPOSE]`
- **Audit Logging**: `[YES/NO — and what actions are logged]`

---

## 5. API Contracts

### Auth Scheme

- **Type**: `[jwt / oauth2 / api_key / mtls / session / combined]`
- **Provider**: `[NAME OF IDENTITY PROVIDER OR "custom"]`
- **Token Lifetime**: `[DURATION — e.g., "15 minutes (access), 30 days (refresh)"]`
- **Refresh Strategy**: `[HOW REFRESH WORKS]`
- **MFA Required**: `[YES/NO — and for which user types]`
- **Scopes**:
  - `[SCOPE NAME]`: `[WHAT IT PERMITS]` — Resources: `[LIST]`

### Versioning Strategy

`[HOW API VERSIONS ARE MANAGED — e.g., "URL path versioning (/v1/, /v2/). Minimum 12-month deprecation notice. Major version increments for breaking changes only."]`

### API Contract: `[API NAME — e.g., "Query API v1"]`

**Base Path**: `[/api/v1/...]`
**Description**: `[WHAT THIS API DOES AND WHO USES IT]`

#### Endpoint: `[METHOD] [PATH]`
- **Summary**: `[ONE LINE DESCRIPTION]`
- **Auth Required**: `[YES/NO + SCOPE REQUIRED]`
- **Request**:
  - Path Params: `[PARAM NAME: TYPE — description]`
  - Query Params: `[PARAM NAME: TYPE — description, required/optional]`
  - Body:
    ```json
    {
      "[FIELD]": "[TYPE — description]",
      "[FIELD]": "[TYPE — description]"
    }
    ```
- **Response (200)**:
  ```json
  {
    "[FIELD]": "[TYPE — description]",
    "[FIELD]": "[TYPE — description]"
  }
  ```
- **Error Codes**: `[ERR-001, ERR-002]`
- **Rate Limit**: `[NUMBER requests per WINDOW per SCOPE]`

`[ADD ENDPOINTS FOR EACH API OPERATION]`

**Error Codes**:
| Code | HTTP | Message | Resolution |
|------|------|---------|-----------|
| `[ERR-001]` | `[4xx/5xx]` | `[USER-FACING MESSAGE]` | `[HOW TO RESOLVE]` |

---

## 6. Security

### Threat Model

**Methodology**: `[STRIDE / PASTA / LINDDUN / custom]`

**Assets**:
| ID | Name | Type | Sensitivity | Description |
|----|------|------|-------------|-------------|
| ASSET-001 | `[NAME]` | `[data/service/credential/infrastructure]` | `[critical/high/medium/low]` | `[WHAT THIS ASSET IS AND WHY IT'S VALUABLE]` |

**Trust Boundaries**:
| ID | Name | Components Inside | Description |
|----|------|------------------|-------------|
| TB-001 | `[NAME — e.g., "Public Internet Boundary"]` | `[COMPONENT IDs]` | `[WHAT THIS BOUNDARY PROTECTS]` |

**Threats**:
| ID | Category | Description | Target | Likelihood | Impact | Mitigations |
|----|----------|-------------|--------|-----------|--------|------------|
| THR-001 | `[STRIDE CATEGORY]` | `[ATTACK SCENARIO]` | `[ASSET-ID]` | `[high/medium/low]` | `[critical/high/medium/low]` | `[SC-IDs]` |

### Security Controls

| ID | Name | Type | Category | Description | Implementation |
|----|------|------|----------|-------------|----------------|
| SC-001 | `[CONTROL NAME]` | `[preventive/detective/corrective]` | `[identity/network/data/application/operational]` | `[WHAT THIS CONTROL DOES]` | `[HOW IT IS IMPLEMENTED TECHNICALLY]` |

### Compliance Requirements

- `[REQUIREMENT 1 — e.g., "SOC 2 Type II: Annual audit required. Controls mapped to CC series."]`
- `[REQUIREMENT 2 — e.g., "GDPR: Data subject rights (access, deletion, portability) must be implemented."]`
- `[...]`

### Risk Register

| ID | Title | Category | Likelihood (1-5) | Impact (1-5) | Score | Mitigation | Owner | Status |
|----|-------|----------|-----------------|-------------|-------|-----------|-------|--------|
| RISK-001 | `[TITLE]` | `[security/operational/compliance/reputational/financial]` | `[1-5]` | `[1-5]` | `[L*I]` | `[HOW IT'S MITIGATED]` | `[WHO OWNS THIS RISK]` | `[open/mitigated/accepted/transferred]` |

---

## 7. Infrastructure

### Cloud Configuration

- **Cloud Provider**: `[AWS / GCP / Azure / Multi-cloud / On-premise — with rationale]`
- **Primary Region**: `[REGION CODE — e.g., "us-east-1"]`
- **Secondary Regions**: `[LIST — for DR or latency-based routing]`

### Environments

| Environment | Purpose | Isolated | Config Notes |
|-------------|---------|----------|-------------|
| `dev` | `[DEVELOPMENT PURPOSE]` | `[YES/NO]` | `[KEY DIFFERENCES FROM PROD]` |
| `staging` | `[STAGING PURPOSE]` | `[YES/NO]` | `[KEY DIFFERENCES FROM PROD]` |
| `production` | `[PROD PURPOSE]` | Yes | — |

### Compute Configuration

| Component | Runtime Type | Technology | Min Instances | Max Instances | CPU | Memory | Autoscale Metric |
|-----------|-------------|-----------|--------------|--------------|-----|--------|-----------------|
| `[COMP-ID]` | `[container/serverless/vm/managed_service]` | `[TECH — e.g., "ECS Fargate"]` | `[N]` | `[N]` | `[vCPU]` | `[GB]` | `[METRIC]` |

### CI/CD Pipeline

- **Platform**: `[GitHub Actions / GitLab CI / Jenkins / CircleCI / ...]`
- **Branch Strategy**: `[e.g., "trunk-based development with short-lived feature branches"]`
- **Deployment Strategy**: `[blue_green / canary / rolling / recreate — with rationale]`
- **Rollback Procedure**: `[HOW TO ROLL BACK A FAILED DEPLOYMENT]`

**Pipeline Stages**:
1. `[STAGE NAME — e.g., "Test"]`: Trigger: `[EVENT]` | Steps: `[LIST]` | Gate: `[PASS CRITERIA]`
2. `[STAGE NAME — e.g., "Build"]`: Trigger: `[EVENT]` | Steps: `[LIST]` | Gate: `[PASS CRITERIA]`
3. `[STAGE NAME — e.g., "Deploy Staging"]`: Trigger: `[EVENT]` | Steps: `[LIST]` | Gate: `[PASS CRITERIA]`
4. `[STAGE NAME — e.g., "Deploy Production"]`: Trigger: `[EVENT]` | Steps: `[LIST]` | Gate: `[PASS CRITERIA — e.g., manual approval + all staging tests green]`

### Observability

- **Metrics**: `[PLATFORM — e.g., "Prometheus + Grafana"]`
- **Logging**: `[PLATFORM — e.g., "ELK Stack / Datadog"]`
- **Tracing**: `[PLATFORM — e.g., "Jaeger / AWS X-Ray"]`

**SLO Definitions**:
| SLO | Target | Window | Indicator | Alert On |
|-----|--------|--------|-----------|---------|
| `[SLO NAME]` | `[TARGET — e.g., 99.9%]` | `[ROLLING 30 DAYS]` | `[WHAT IS MEASURED]` | `[BURN RATE THRESHOLD]` |

**Key Alert Rules**:
| Alert | Condition | Severity | Channel | Runbook |
|-------|-----------|----------|---------|---------|
| `[ALERT NAME]` | `[TRIGGER CONDITION]` | `[critical/warning/info]` | `[SLACK/PAGERDUTY/...]` | `[LINK OR PATH TO RUNBOOK]` |

### Disaster Recovery

- **RTO**: `[e.g., "4 hours" — maximum acceptable downtime]`
- **RPO**: `[e.g., "1 hour" — maximum acceptable data loss]`
- **Strategy**: `[active_passive / active_active / pilot_light / backup_restore]`
- **Backup Frequency**: `[SCHEDULE]`
- **Testing Schedule**: `[HOW OFTEN DR IS TESTED — e.g., "Quarterly failover test"]`

---

## 8. Implementation Plan

### Tech Stack

| Layer | Technology | Version | Purpose | Rationale |
|-------|-----------|---------|---------|-----------|
| `[Backend language]` | `[NAME]` | `[VERSION]` | `[WHY IN STACK]` | `[WHY CHOSEN OVER ALTERNATIVES]` |
| `[Frontend]` | `[NAME]` | `[VERSION]` | `[WHY IN STACK]` | `[WHY CHOSEN]` |
| `[Primary database]` | `[NAME]` | `[VERSION]` | `[WHY IN STACK]` | `[WHY CHOSEN]` |
| `[Message broker]` | `[NAME]` | `[VERSION]` | `[WHY IN STACK]` | `[WHY CHOSEN]` |
| `[Cache]` | `[NAME]` | `[VERSION]` | `[WHY IN STACK]` | `[WHY CHOSEN]` |
| `[Container runtime]` | `[NAME]` | `[VERSION]` | `[WHY IN STACK]` | `[WHY CHOSEN]` |
| `[...]` | `[...]` | `[...]` | `[...]` | `[...]` |

### Team Structure

- **Total Headcount**: `[NUMBER]`

| Role | Count | Responsibilities | Required Skills |
|------|-------|-----------------|----------------|
| `[ROLE TITLE]` | `[N]` | `[LIST OF KEY RESPONSIBILITIES]` | `[LIST OF REQUIRED SKILLS]` |

### Build Phases

#### Phase 1: `[PHASE NAME — e.g., "Core Foundation"]`
- **Objective**: `[WHAT THIS PHASE ACHIEVES AND WHY IT'S FIRST]`
- **Duration**: `[N weeks]`
- **Dependencies**: None (first phase) OR `[LIST PHASE IDs THAT MUST COMPLETE FIRST]`
- **Deliverables**:
  - `[DELIVERABLE NAME]`: `[DESCRIPTION]` — Done when: `[SPECIFIC MEASURABLE CRITERIA]`
  - `[DELIVERABLE NAME]`: `[DESCRIPTION]` — Done when: `[SPECIFIC MEASURABLE CRITERIA]`
- **Risks**: `[PHASE-SPECIFIC RISKS AND HOW THEY'RE MANAGED]`

#### Phase 2: `[PHASE NAME]`
`[REPEAT STRUCTURE ABOVE]`

#### Phase N: `[PHASE NAME]`
`[REPEAT STRUCTURE ABOVE]`

### Milestones

| ID | Name | Target Date | Phase | Acceptance Criteria | Stakeholders |
|----|------|------------|-------|--------------------|--------------|
| MS-01 | `[MILESTONE NAME]` | `[DATE]` | `[PHASE ID]` | `[SPECIFIC, MEASURABLE CRITERIA THAT MUST ALL BE TRUE]` | `[WHO SIGNS OFF]` |

### Effort Estimate

- **Total Duration**: `[N weeks]`
- **Headcount Assumption**: `[N engineers]`
- **Confidence**: `[high / medium / low]`
- **Methodology**: `[HOW ESTIMATE WAS DERIVED — e.g., "Function point analysis, reference class forecasting against 3 similar projects"]`
- **Risk Buffer**: `[N% — e.g., "20% buffer applied to account for integration complexity"]`

| Phase | Estimate | Notes |
|-------|---------|-------|
| Phase 1 | `[N weeks]` | `[KEY ASSUMPTIONS]` |
| Phase 2 | `[N weeks]` | `[KEY ASSUMPTIONS]` |
| Phase N | `[N weeks]` | `[KEY ASSUMPTIONS]` |

---

## 9. Quality Standards

### Test Strategy

| Test Type | Framework | Scope | Trigger | Pass Criteria |
|-----------|----------|-------|---------|--------------|
| Unit | `[FRAMEWORK]` | `[WHAT'S TESTED]` | `[EVERY COMMIT]` | `[COVERAGE + PASS RATE]` |
| Integration | `[FRAMEWORK]` | `[WHAT'S TESTED]` | `[EVERY PR]` | `[CRITERIA]` |
| E2E | `[FRAMEWORK]` | `[WHAT'S TESTED]` | `[PRE-DEPLOY]` | `[CRITERIA]` |
| Performance | `[FRAMEWORK]` | `[LOAD PROFILE]` | `[PRE-RELEASE]` | `[LATENCY + ERROR RATE TARGETS]` |
| Security | `[TOOL]` | `[SAST/DAST/DEPS]` | `[SCHEDULE/PRE-RELEASE]` | `[NO HIGH/CRITICAL VULNS]` |

**Coverage Target**: `[N%]`

### Non-Functional Requirements

| NFR | Target | Measurement | Notes |
|-----|--------|-------------|-------|
| Availability | `[e.g., 99.9%]` | `[HOW MEASURED]` | `[EXCLUSIONS]` |
| Latency (p50) | `[N ms]` | `[HOW MEASURED]` | `[AT WHAT LOAD]` |
| Latency (p99) | `[N ms]` | `[HOW MEASURED]` | `[AT WHAT LOAD]` |
| Throughput | `[N RPS]` | `[HOW MEASURED]` | `[SUSTAINED VS PEAK]` |
| Error Rate | `[< N%]` | `[HOW MEASURED]` | `[5xx rate]` |
| MTTR | `[< N hours]` | `[HOW MEASURED]` | `[P1 incidents]` |

### Definition of Done

<!-- These criteria apply globally — every milestone must satisfy all of them. -->

- [ ] All acceptance criteria for the milestone are verifiably met
- [ ] Unit test coverage >= `[N%]` for all new code
- [ ] All integration tests passing in CI
- [ ] All E2E tests passing in staging environment
- [ ] No P0 or P1 bugs open
- [ ] Security scan passing — no high or critical vulnerabilities
- [ ] Performance tests passing against NFR targets
- [ ] API documentation updated and accurate
- [ ] Runbooks written for all new operational concerns
- [ ] Observability: dashboards live, alerts configured, on-call notified
- [ ] `[ADD PROJECT-SPECIFIC CRITERIA]`

---

## 10. Handoff Configuration

### Target Systems

| Priority | System | Type | Endpoint/Config | Notes |
|----------|--------|------|----------------|-------|
| 1 | `[TARGET NAME]` | `[claude-code/github/linear/jira/langchain/autogen/crewai/generic-api]` | `[ENDPOINT OR CONFIG]` | `[SPECIAL INSTRUCTIONS]` |

### Export Formats

`[LIST: json, markdown, yaml, claude_md, openapi, terraform — select all that apply]`

### Delivery Method

`[file / api / clipboard / git_commit]`

---

## Changelog

| Version | Date | Author | Summary |
|---------|------|--------|---------|
| 1.0.0 | `[DATE]` | `[AUTHOR]` | Initial blueprint creation |

---

*Blueprint produced by SuperArchitect OS. Schema version: 1.0.0.*
