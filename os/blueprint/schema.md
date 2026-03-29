# System Blueprint Schema

> The complete, formal data contract for a System Blueprint. Every field, type, description, and example value defined. This schema is the source of truth for blueprint validation, serialization, and handoff transformation.

---

## Root Schema: `Blueprint`

```typescript
Blueprint {
  meta:           Meta
  problem:        Problem
  system:         System
  architecture:   Architecture
  data:           Data
  api:            API
  security:       Security
  infrastructure: Infrastructure
  implementation: Implementation
  quality:        Quality
  handoff:        Handoff
}
```

---

## `Meta`

Identity and provenance of the blueprint.

```typescript
Meta {
  id:                   string (UUID v4)
  name:                 string
  version:              string (semver: "MAJOR.MINOR.PATCH")
  created_at:           string (ISO 8601: "2026-03-28T14:30:00Z")
  updated_at:           string (ISO 8601)
  status:               "draft" | "review" | "approved" | "active" | "deprecated"
  architect:            string
  approved_by?:         string
  approved_at?:         string (ISO 8601)
  tags:                 string[]
  parent_blueprint_id?: string (UUID v4)
  changelog:            ChangelogEntry[]
}
```

**Field Definitions:**

| Field | Type | Required | Description | Example |
|-------|------|----------|-------------|---------|
| `id` | UUID | Yes | Globally unique blueprint identifier | `"a3f8c2d1-4b5e-4f6a-8c9d-0e1f2a3b4c5d"` |
| `name` | string | Yes | Human-readable blueprint name | `"Multi-tenant SaaS Analytics Platform"` |
| `version` | semver | Yes | Semantic version of this blueprint | `"1.0.0"` |
| `created_at` | ISO 8601 | Yes | When the blueprint was first created | `"2026-03-28T14:30:00Z"` |
| `updated_at` | ISO 8601 | Yes | When the blueprint was last modified | `"2026-03-28T16:45:00Z"` |
| `status` | enum | Yes | Current lifecycle stage | `"approved"` |
| `architect` | string | Yes | Name of the architect or "SuperArchitect OS" | `"SuperArchitect OS v1.0"` |
| `approved_by` | string | No | Who approved the blueprint | `"Jane Smith, Chief Architect"` |
| `approved_at` | ISO 8601 | No | When approval was granted | `"2026-03-28T17:00:00Z"` |
| `tags` | string[] | Yes | Searchable tags for registry indexing | `["fintech", "api", "event-driven"]` |
| `parent_blueprint_id` | UUID | No | ID of the blueprint this extends | `"b4a9d3e2-..."` |
| `changelog` | ChangelogEntry[] | Yes | Version history (empty array for v1.0.0) | See below |

### `ChangelogEntry`

```typescript
ChangelogEntry {
  version:    string (semver)
  date:       string (ISO 8601)
  author:     string
  summary:    string
  changes:    string[]
}
```

---

## `Problem`

The validated problem definition. Anchors all architectural decisions.

```typescript
Problem {
  statement:        string
  context:          string
  constraints:      Constraint[]
  assumptions:      Assumption[]
  success_criteria: SuccessCriterion[]
}
```

### `Constraint`

```typescript
Constraint {
  id:       string ("CON-001", "CON-002", ...)
  type:     "technical" | "business" | "regulatory" | "resource" | "time"
  title:    string
  detail:   string
  impact:   "blocking" | "high" | "medium" | "low"
}
```

**Example:**
```json
{
  "id": "CON-001",
  "type": "regulatory",
  "title": "PCI DSS Compliance Required",
  "detail": "The system processes payment card data and must comply with PCI DSS Level 1 requirements, including network segmentation, encryption at rest and in transit, and annual QSA audit.",
  "impact": "blocking"
}
```

### `Assumption`

```typescript
Assumption {
  id:           string ("ASM-001", "ASM-002", ...)
  statement:    string
  risk_if_wrong: string
  owner:        string
}
```

### `SuccessCriterion`

```typescript
SuccessCriterion {
  id:         string ("SC-001", "SC-002", ...)
  title:      string
  metric:     string
  target:     string
  measurement: string
  timeframe:  string
}
```

**Example:**
```json
{
  "id": "SC-001",
  "title": "Dashboard Query Latency",
  "metric": "p99 query response time",
  "target": "< 500ms",
  "measurement": "APM traces, percentile aggregation over 5-minute windows",
  "timeframe": "At 70% of peak load within 30 days of production launch"
}
```

---

## `System`

High-level system identity and scope.

```typescript
System {
  name:          string
  description:   string
  domain:        string
  type:          SystemType
  users:         UserPersona[]
  scale_targets: ScaleTarget
}

SystemType =
  "web_app" | "api" | "data_platform" | "ai_system" | "cli" |
  "library" | "sdk" | "microservices_platform" | "event_pipeline" |
  "ml_platform" | "iot_backend" | "mobile_backend" | "hybrid"
```

### `UserPersona`

```typescript
UserPersona {
  id:              string ("UP-001", ...)
  name:            string
  role:            string
  goals:           string[]
  pain_points:     string[]
  technical_level: "non-technical" | "semi-technical" | "technical" | "expert"
  usage_frequency: "real-time" | "daily" | "weekly" | "occasional"
  critical_flows:  string[]
}
```

**Example:**
```json
{
  "id": "UP-001",
  "name": "Data Analyst",
  "role": "Builds and monitors business dashboards",
  "goals": ["Real-time visibility into KPIs", "Self-service query exploration", "Shareable reports"],
  "pain_points": ["Slow SQL queries", "No alerting on metric anomalies"],
  "technical_level": "semi-technical",
  "usage_frequency": "daily",
  "critical_flows": ["Create dashboard", "Set metric alert", "Export report to PDF"]
}
```

### `ScaleTarget`

```typescript
ScaleTarget {
  users_initial:    number
  users_peak:       number
  requests_per_second_peak: number
  data_volume_gb_initial:   number
  data_volume_gb_annual_growth: number
  uptime_target_percent:    number
  latency_p99_ms:           number
  geographic_regions:       string[]
}
```

---

## `Architecture`

The core structural definition of the system.

```typescript
Architecture {
  style:                ArchitectureStyle
  style_rationale:      string
  components:           Component[]
  interfaces:           Interface[]
  data_flows:           DataFlow[]
  external_dependencies: Dependency[]
  adrs:                 ADR[]
}

ArchitectureStyle =
  "microservices" | "monolith" | "modular_monolith" | "event_driven" |
  "serverless" | "cqrs_event_sourcing" | "layered" | "hexagonal" |
  "space_based" | "service_mesh"
```

### `Component`

```typescript
Component {
  id:               string ("COMP-001", ...)
  name:             string
  type:             ComponentType
  responsibility:   string
  owns:             string[]          // Data entities this component owns
  exposes:          string[]          // Interfaces/APIs this component exposes
  consumes:         string[]          // Interfaces/APIs this component consumes
  tech_stack:       string[]
  scaling_strategy: "horizontal" | "vertical" | "auto" | "fixed"
  criticality:      "critical" | "high" | "medium" | "low"
  team_owner:       string
}

ComponentType =
  "service" | "gateway" | "worker" | "scheduler" | "cache" |
  "database" | "message_broker" | "cdn" | "load_balancer" |
  "frontend" | "cli" | "sdk" | "ml_model" | "data_pipeline"
```

**Example:**
```json
{
  "id": "COMP-003",
  "name": "query-service",
  "type": "service",
  "responsibility": "Executes analytical queries against ClickHouse on behalf of authenticated tenants. Applies tenant isolation filters, enforces query timeouts, and caches frequent queries in Redis.",
  "owns": ["QueryResult", "QueryCache"],
  "exposes": ["query-api-v1"],
  "consumes": ["storage-api-v1", "auth-service-jwt"],
  "tech_stack": ["Go 1.22", "ClickHouse Go driver", "Redis 7"],
  "scaling_strategy": "horizontal",
  "criticality": "critical",
  "team_owner": "data-platform-team"
}
```

### `Interface`

```typescript
Interface {
  id:       string ("INT-001", ...)
  name:     string
  type:     "rest_api" | "grpc" | "graphql" | "event" | "websocket" | "sdk" | "cli"
  protocol: string
  version:  string
  producer: string   // Component ID
  consumers: string[] // Component IDs
  contract_ref: string // Reference to api.contracts[*].id
  sla: InterfaceSLA
}

InterfaceSLA {
  availability_percent: number
  latency_p99_ms:       number
  throughput_rps:       number
}
```

### `DataFlow`

```typescript
DataFlow {
  id:          string ("DF-001", ...)
  name:        string
  description: string
  trigger:     string
  steps:       DataFlowStep[]
  data_types:  string[]
  pii_involved: boolean
}

DataFlowStep {
  sequence:    number
  from:        string  // Component ID
  to:          string  // Component ID
  via:         string  // Interface ID
  payload:     string  // Description or schema reference
  async:       boolean
}
```

### `Dependency`

```typescript
Dependency {
  id:       string ("DEP-001", ...)
  name:     string
  type:     "saas" | "open_source" | "internal_service" | "data_provider" | "hardware"
  purpose:  string
  version:  string
  license?: string
  risk:     "critical" | "high" | "medium" | "low"
  fallback?: string
}
```

### `ADR` (Architecture Decision Record)

```typescript
ADR {
  id:           string ("ADR-001", ...)
  title:        string
  date:         string (ISO 8601)
  status:       "proposed" | "accepted" | "superseded" | "deprecated"
  context:      string
  decision:     string
  rationale:    string
  alternatives: ADRAlternative[]
  consequences: string[]
  superseded_by?: string  // ADR ID
}

ADRAlternative {
  name:    string
  pros:    string[]
  cons:    string[]
  rejected_because: string
}
```

---

## `Data`

Complete data architecture specification.

```typescript
Data {
  models:     DataModel[]
  stores:     DataStore[]
  pipelines:  Pipeline[]
  governance: DataGovernance
}
```

### `DataModel`

```typescript
DataModel {
  id:          string ("DM-001", ...)
  entity:      string
  description: string
  owner:       string  // Component ID
  fields:      DataField[]
  indexes:     string[]
  relationships: Relationship[]
}

DataField {
  name:        string
  type:        string  // Primitive or reference type
  required:    boolean
  unique:      boolean
  pii:         boolean
  encrypted:   boolean
  description: string
}

Relationship {
  type:        "one_to_one" | "one_to_many" | "many_to_many"
  entity:      string
  foreign_key: string
  cascade:     boolean
}
```

### `DataStore`

```typescript
DataStore {
  id:               string ("DS-001", ...)
  name:             string
  type:             "relational" | "document" | "key_value" | "time_series" | "graph" | "vector" | "object" | "search"
  technology:       string
  version:          string
  purpose:          string
  access_patterns:  string[]
  estimated_size_gb: number
  backup_frequency: string
  replication:      string
}
```

### `Pipeline`

```typescript
Pipeline {
  id:          string ("PIP-001", ...)
  name:        string
  type:        "batch" | "streaming" | "micro_batch"
  trigger:     "schedule" | "event" | "api" | "continuous"
  schedule?:   string (cron)
  source:      PipelineEndpoint
  sink:        PipelineEndpoint
  transforms:  string[]
  sla_latency: string
  technology:  string
}

PipelineEndpoint {
  store_id: string  // DataStore ID
  format:   string
  schema:   string
}
```

### `DataGovernance`

```typescript
DataGovernance {
  data_owner:         string
  classification:     DataClassification[]
  retention_policies: RetentionPolicy[]
  access_controls:    DataAccessControl[]
  pii_fields:         string[]
  anonymization_strategy: string
  audit_logging:      boolean
}

DataClassification {
  level:       "public" | "internal" | "confidential" | "restricted"
  description: string
  examples:    string[]
  controls:    string[]
}

RetentionPolicy {
  data_type: string
  duration:  string
  rationale: string
}
```

---

## `API`

All API contracts and meta-configuration.

```typescript
API {
  contracts:          APIContract[]
  auth_scheme:        AuthScheme
  versioning_strategy: string
}
```

### `APIContract`

```typescript
APIContract {
  id:          string ("API-001", ...)
  name:        string
  type:        "rest" | "grpc" | "graphql" | "webhook" | "async"
  base_path:   string
  version:     string
  description: string
  endpoints:   Endpoint[]
  error_codes: ErrorCode[]
  rate_limits: RateLimit[]
}

Endpoint {
  method:      "GET" | "POST" | "PUT" | "PATCH" | "DELETE" | "SUBSCRIBE"
  path:        string
  summary:     string
  auth:        boolean
  request:     RequestSchema
  response:    ResponseSchema
  errors:      string[]  // Error code IDs
}

RequestSchema {
  path_params?:  Record<string, FieldSchema>
  query_params?: Record<string, FieldSchema>
  headers?:      Record<string, FieldSchema>
  body?:         ObjectSchema
}

ResponseSchema {
  "200"?: ObjectSchema
  "201"?: ObjectSchema
  "204"?: null
  "400"?: ErrorSchema
  "401"?: ErrorSchema
  "403"?: ErrorSchema
  "404"?: ErrorSchema
  "429"?: ErrorSchema
  "500"?: ErrorSchema
}

FieldSchema {
  type:        string
  required:    boolean
  description: string
  example:     any
}

ErrorCode {
  code:        string
  http_status: number
  message:     string
  resolution:  string
}

RateLimit {
  scope:       "global" | "per_user" | "per_tenant" | "per_ip"
  requests:    number
  window:      string
  enforcement: "hard" | "soft"
}
```

### `AuthScheme`

```typescript
AuthScheme {
  type:           "jwt" | "oauth2" | "api_key" | "mtls" | "session" | "combined"
  provider:       string
  token_lifetime: string
  refresh_strategy: string
  scopes:         AuthScope[]
  mfa_required:   boolean
}

AuthScope {
  name:        string
  description: string
  resources:   string[]
  actions:     string[]
}
```

---

## `Security`

Complete security specification.

```typescript
Security {
  threat_model:            ThreatModel
  controls:                SecurityControl[]
  compliance_requirements: string[]
  risk_register:           Risk[]
}
```

### `ThreatModel`

```typescript
ThreatModel {
  methodology:  "STRIDE" | "PASTA" | "LINDDUN" | "custom"
  assets:       Asset[]
  threats:      Threat[]
  trust_boundaries: TrustBoundary[]
}

Asset {
  id:               string ("ASSET-001", ...)
  name:             string
  type:             "data" | "service" | "credential" | "infrastructure"
  sensitivity:      "critical" | "high" | "medium" | "low"
  description:      string
}

Threat {
  id:          string ("THR-001", ...)
  category:    "Spoofing" | "Tampering" | "Repudiation" | "Information Disclosure" | "Denial of Service" | "Elevation of Privilege"
  description: string
  target:      string  // Asset ID
  likelihood:  "high" | "medium" | "low"
  impact:      "critical" | "high" | "medium" | "low"
  mitigations: string[]  // SecurityControl IDs
}

TrustBoundary {
  id:          string ("TB-001", ...)
  name:        string
  description: string
  components:  string[]  // Component IDs inside this boundary
}
```

### `SecurityControl`

```typescript
SecurityControl {
  id:           string ("SC-001", ...)
  name:         string
  type:         "preventive" | "detective" | "corrective" | "deterrent"
  category:     "identity" | "network" | "data" | "application" | "operational"
  description:  string
  implementation: string
  addresses:    string[]  // Threat IDs
  tested_by:    string
}
```

### `Risk`

```typescript
Risk {
  id:          string ("RISK-001", ...)
  title:       string
  description: string
  category:    "security" | "operational" | "compliance" | "reputational" | "financial"
  likelihood:  1 | 2 | 3 | 4 | 5
  impact:      1 | 2 | 3 | 4 | 5
  risk_score:  number  // likelihood * impact
  mitigation:  string
  owner:       string
  status:      "open" | "mitigated" | "accepted" | "transferred"
}
```

---

## `Infrastructure`

Everything needed to run the system in production.

```typescript
Infrastructure {
  cloud_provider:    string
  regions:           string[]
  topology:          InfraTopology
  ci_cd:             CICDPlan
  observability:     ObservabilityPlan
  disaster_recovery: DRPlan
}
```

### `InfraTopology`

```typescript
InfraTopology {
  environments:  Environment[]
  network:       NetworkConfig
  compute:       ComputeConfig[]
  storage:       StorageConfig[]
  secrets:       SecretsConfig
}

Environment {
  name:     "dev" | "staging" | "production" | "dr"
  purpose:  string
  isolated: boolean
  config_overrides: Record<string, string>
}

NetworkConfig {
  vpc_cidr:     string
  subnets:      Subnet[]
  load_balancers: LoadBalancer[]
  firewall_rules: FirewallRule[]
}

ComputeConfig {
  component_id:  string
  type:          "container" | "serverless" | "vm" | "managed_service"
  runtime:       string
  min_instances: number
  max_instances: number
  cpu:           string
  memory:        string
  autoscale_metric: string
}
```

### `CICDPlan`

```typescript
CICDPlan {
  platform:     string
  pipeline_stages: PipelineStage[]
  branch_strategy: string
  deployment_strategy: "blue_green" | "canary" | "rolling" | "recreate"
  rollback_procedure: string
}

PipelineStage {
  name:     string
  trigger:  string
  steps:    string[]
  gates:    string[]
  timeout:  string
}
```

### `ObservabilityPlan`

```typescript
ObservabilityPlan {
  metrics_platform:  string
  logging_platform:  string
  tracing_platform:  string
  dashboards:        Dashboard[]
  alert_rules:       AlertRule[]
  slo_definitions:   SLO[]
}

SLO {
  name:       string
  target:     string
  window:     string
  indicator:  string
  alert_on:   string
}

AlertRule {
  name:       string
  condition:  string
  severity:   "critical" | "warning" | "info"
  channel:    string
  runbook:    string
}
```

### `DRPlan`

```typescript
DRPlan {
  rto:              string  // Recovery Time Objective
  rpo:              string  // Recovery Point Objective
  strategy:         "active_passive" | "active_active" | "pilot_light" | "backup_restore"
  backup_frequency: string
  failover_steps:   string[]
  testing_schedule: string
}
```

---

## `Implementation`

The sequenced build plan.

```typescript
Implementation {
  phases:         Phase[]
  milestones:     Milestone[]
  team_structure: TeamStructure
  tech_stack:     TechStack
  effort_estimate: EffortEstimate
}
```

### `Phase`

```typescript
Phase {
  id:           string ("PH-01", ...)
  name:         string
  objective:    string
  duration:     string
  sequence:     number
  dependencies: string[]  // Phase IDs that must complete first
  deliverables: Deliverable[]
  risks:        string[]
}

Deliverable {
  id:          string
  name:        string
  description: string
  type:        "service" | "feature" | "infrastructure" | "documentation" | "test_suite"
  definition_of_done: string[]
}
```

### `Milestone`

```typescript
Milestone {
  id:                  string ("MS-01", ...)
  name:                string
  target_date:         string (ISO 8601)
  phase:               string  // Phase ID
  acceptance_criteria: string[]
  stakeholders:        string[]
}
```

### `TeamStructure`

```typescript
TeamStructure {
  total_headcount: number
  roles:           Role[]
  squads?:         Squad[]
}

Role {
  title:           string
  count:           number
  responsibilities: string[]
  required_skills: string[]
}

Squad {
  name:       string
  focus:      string
  members:    string[]  // Role titles
  components: string[]  // Component IDs they own
}
```

### `TechStack`

```typescript
TechStack {
  languages:   TechChoice[]
  frameworks:  TechChoice[]
  databases:   TechChoice[]
  messaging:   TechChoice[]
  infra_tools: TechChoice[]
  dev_tools:   TechChoice[]
}

TechChoice {
  name:     string
  version:  string
  purpose:  string
  rationale: string
  adr_ref?: string  // ADR ID if decision was formally recorded
}
```

### `EffortEstimate`

```typescript
EffortEstimate {
  total_weeks:        number
  confidence:         "high" | "medium" | "low"
  methodology:        string
  by_phase:           Record<string, string>  // Phase ID → estimate
  headcount_assumption: number
  risk_buffer_percent: number
}
```

---

## `Quality`

The quality covenant.

```typescript
Quality {
  test_strategy:      TestStrategy
  nfr_targets:        NFRTargets
  definition_of_done: string[]
}
```

### `TestStrategy`

```typescript
TestStrategy {
  unit:         TestConfig
  integration:  TestConfig
  e2e:          TestConfig
  performance:  PerformanceTestConfig
  security:     SecurityTestConfig
  coverage_target_percent: number
}

TestConfig {
  framework:    string
  scope:        string
  trigger:      string
  pass_criteria: string
}

PerformanceTestConfig extends TestConfig {
  load_profile:   string
  duration:       string
  success_criteria: Record<string, string>
}

SecurityTestConfig extends TestConfig {
  tools:      string[]
  scan_types: string[]
}
```

### `NFRTargets`

```typescript
NFRTargets {
  availability:      string   // "99.9%"
  latency_p50_ms:    number
  latency_p99_ms:    number
  throughput_rps:    number
  error_rate_max:    string   // "0.1%"
  recovery_time:     string
  mttr:              string   // Mean Time to Recovery
}
```

---

## `Handoff`

Handoff configuration.

```typescript
Handoff {
  target_systems:  TargetSystem[]
  export_formats:  ExportFormat[]
  delivery_method: "file" | "api" | "clipboard" | "git_commit"
}

## Unified with Canonical System Package

The System Blueprint IS the canonical system package. The Blueprint Protocol v2.0 adopts the kernel's package schema as its foundation.

### Additional Fields (v2.0)

The following fields are added to the blueprint schema to support kernel integration:

- `candidates` — Array of candidate architectures considered (not just the winner)
- `selection` — Why the chosen architecture won (rationale, confidence, hybridization details)
- `audit` — Machine-readable audit vectors with scores, evidence, uncertainty, deltas
- `routing_state` — Current kernel phase and routing status
- `evolution` — Immutable ledger of all reroutes and mutations during the build
- `success_model` — Project-specific definition of "world class" with measurable thresholds

These fields ensure the blueprint is not just a final document but a complete record of the architectural reasoning process.

TargetSystem {
  id:       string
  type:     "claude-code" | "github" | "linear" | "jira" | "langchain" | "autogen" | "crewai" | "generic-api"
  name:     string
  endpoint?: string
  config:   Record<string, string>
  priority: number
}

ExportFormat = "json" | "markdown" | "yaml" | "claude_md" | "openapi" | "terraform"
```
