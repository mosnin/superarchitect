# Handoff Adapter: GitHub

> Transform a System Blueprint into a complete, immediately operational GitHub project — repository structure, issues, milestones, project board, CI/CD pipelines, branch protections, and team configuration.

---

## Overview

The GitHub adapter turns a blueprint into a living project in minutes. When this handoff completes, the engineering team has a GitHub repository that reflects the system's architecture, a fully-populated issue backlog that maps to the implementation plan, a project board organized by phase, and CI/CD pipelines derived from the infrastructure specification.

The adapter uses the GitHub CLI (`gh`) for all operations. Every command is deterministic and idempotent — running the handoff twice produces the same result.

---

## Blueprint → GitHub Mapping

| Blueprint Section | GitHub Artifact |
|-------------------|----------------|
| `meta.name` + `system.description` | Repository name + description |
| `architecture.components` | Repository directory structure + CODEOWNERS |
| `implementation.phases` | Milestones + Project board columns |
| `implementation.phases[*].deliverables` | GitHub Issues (grouped by milestone) |
| `implementation.team_structure.roles` | Issue labels by team/role |
| `quality.definition_of_done` | PR template checklist |
| `quality.test_strategy` | CI workflow (test stage) |
| `infrastructure.ci_cd` | GitHub Actions workflows |
| `security.compliance_requirements` | Repository security settings |
| `architecture.adrs` | Issues labeled `adr` in `docs/architecture/` |
| `api.contracts` | Issues labeled `api-contract` |

---

## Repository Structure

The adapter creates the repository with a directory structure that mirrors the blueprint's component architecture.

**Structure generation rules:**
- Each `component.type == "service"` → `services/{component.name}/`
- Each `component.type == "frontend"` → `web/{component.name}/`
- Each `component.type == "data_pipeline"` → `pipelines/{component.name}/`
- Infrastructure config → `infra/`
- API contracts → `docs/api/`
- ADRs → `docs/architecture/decisions/`
- Runbooks → `docs/runbooks/`

---

## GitHub Issues from Implementation Tasks

Each deliverable in each phase becomes a GitHub Issue. Issues are structured for maximum actionability.

**Issue title format**: `[{phase.id}] {deliverable.name}`

**Issue body template:**
```markdown
## Objective
{deliverable.description}

## Acceptance Criteria
{deliverable.definition_of_done mapped as checkbox list}

## Phase
{phase.name} — {phase.objective}

## Components Affected
{relevant component IDs and names}

## Definition of Done
{quality.definition_of_done applicable to this type of deliverable}

---
*Generated from Blueprint: {meta.name} v{meta.version}*
*Blueprint ID: {meta.id}*
```

**Labels applied to every issue:**
- Phase label: `phase-1`, `phase-2`, etc.
- Type label: `service`, `feature`, `infrastructure`, `documentation`, `test-suite`
- Priority label: derived from `deliverable.criticality` if present, else `medium`
- Component label: `comp:{component.name}`

---

## GitHub Projects Board

The project board uses GitHub Projects v2. Phases become columns (or iterations). Each issue is placed in its corresponding phase column.

**Board columns:**
- `Backlog` — Issues not yet in an active phase
- `Phase 1: {phase.name}` — Phase 1 issues
- `Phase 2: {phase.name}` — Phase 2 issues
- ...
- `Phase N: {phase.name}`
- `In Progress` — Currently active issues
- `Review` — PRs open for these issues
- `Done` — Completed issues

**Board views:**
- **By Phase** (default): Issues grouped by phase column
- **By Component**: Issues grouped by component label
- **By Assignee**: Issues grouped by team member
- **Roadmap**: Milestone timeline view

---

## GitHub Actions CI/CD

The adapter generates workflow files from `infrastructure.ci_cd`. These are production-ready, not skeleton files.

### Workflow: `ci.yml` (Continuous Integration)

```yaml
# Generated from blueprint: {meta.name} v{meta.version}
# infrastructure.ci_cd.pipeline_stages → test + build stages

name: CI

on:
  push:
    branches: [main, 'feature/**', 'fix/**']
  pull_request:
    branches: [main]

jobs:
  test:
    name: Test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      # Steps generated from infrastructure.ci_cd.pipeline_stages[type=="test"]
      {generated test steps based on tech stack}

      - name: Upload coverage
        uses: codecov/codecov-action@v4
        with:
          threshold: {quality.test_strategy.coverage_target_percent}%

  security-scan:
    name: Security Scan
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      # Generated from security.controls + quality.test_strategy.security
      {generated security scan steps based on compliance requirements}

  build:
    name: Build
    needs: [test, security-scan]
    runs-on: ubuntu-latest
    steps:
      {generated build steps based on tech stack}
```

### Workflow: `deploy-staging.yml`

```yaml
# Generated from infrastructure.ci_cd — staging deployment stage

name: Deploy to Staging

on:
  push:
    branches: [main]

jobs:
  deploy:
    name: Deploy Staging
    environment: staging
    runs-on: ubuntu-latest
    steps:
      {generated deployment steps based on infrastructure.topology and cloud_provider}
```

### Workflow: `deploy-production.yml`

```yaml
# Generated from infrastructure.ci_cd — production deployment stage
# Deployment strategy: {infrastructure.ci_cd.deployment_strategy}

name: Deploy to Production

on:
  workflow_dispatch:
    inputs:
      confirm:
        description: 'Type DEPLOY to confirm'
        required: true

jobs:
  deploy:
    name: Deploy Production
    environment: production  # Requires manual approval in GitHub
    if: github.event.inputs.confirm == 'DEPLOY'
    runs-on: ubuntu-latest
    steps:
      {generated deployment steps with blue/green or canary logic}
```

---

## Branch Protection Rules

Rules are derived from `quality.definition_of_done` and `infrastructure.ci_cd`.

**Rules applied to `main`:**
- Require pull request reviews: 1 approving review minimum
- Dismiss stale pull request approvals when new commits are pushed
- Require status checks to pass before merging: `test`, `security-scan`, `build`
- Require branches to be up to date before merging
- Require linear history
- Include administrators (no bypassing)

---

## CODEOWNERS

Generated from `implementation.team_structure.squads` (or `roles` if no squads defined).

```
# CODEOWNERS generated from blueprint: {meta.name}
# Team structure: {implementation.team_structure}

# Global fallback — all changes require tech lead review
* @{tech-lead-github-handle}

{for each squad:}
# {squad.name} — owns {squad.components joined with ", "}
{for each component in squad.components:}
/{component-directory}/ @{squad.github-team-slug}
{end for}
{end for}

# Documentation
/docs/ @{tech-lead-github-handle}

# Infrastructure
/infra/ @{platform-team-github-handle}
/.github/ @{platform-team-github-handle}
```

---

## PR Template

Generated from `quality.definition_of_done` + `quality.test_strategy`.

```markdown
<!-- Generated from blueprint: {meta.name} — quality.definition_of_done -->

## What does this PR do?
<!-- One to three sentences describing the change and why it's needed -->

## Blueprint Reference
<!-- Which deliverable(s) does this close? e.g., "Closes #42 (DEL-003: JWT Middleware)" -->
Closes #

## Definition of Done Checklist
<!-- Every item must be checked before requesting review -->

{quality.definition_of_done mapped as checkbox list}

## Test Evidence
<!-- Paste test output or link to CI run -->

## Security Considerations
<!-- Does this change affect any security controls? If yes, describe how the control is maintained. -->
- [ ] No security controls affected
- [ ] Security controls affected — described below:

## Rollback Plan
<!-- How do we undo this if it causes problems in production? -->
```

---

## Complete Example: Blueprint → GitHub Commands

The following `gh` commands, when executed in order, set up the complete project from a blueprint. This is the exact output of the GitHub adapter for a hypothetical "Real-time Analytics API" blueprint.

```bash
#!/bin/bash
# GitHub Handoff Script
# Generated from: Real-time Analytics API v1.0.0
# Blueprint ID: a3f8c2d1-4b5e-4f6a-8c9d-0e1f2a3b4c5d
# Generated: 2026-03-28T17:30:00Z

set -euo pipefail

REPO_NAME="realtime-analytics-api"
ORG="your-org"
BLUEPRINT_VERSION="1.0.0"

echo "=== SuperArchitect OS: GitHub Handoff ==="
echo "Blueprint: Real-time Analytics API v${BLUEPRINT_VERSION}"
echo ""

# Step 1: Create Repository
echo "Creating repository..."
gh repo create "${ORG}/${REPO_NAME}" \
  --description "Real-time analytics API with sub-second query latency for 10M+ events/day" \
  --private \
  --clone

cd "${REPO_NAME}"

# Step 2: Initialize directory structure
echo "Initializing directory structure..."
mkdir -p \
  services/ingestion-service/{src,tests} \
  services/query-service/{src,tests} \
  services/auth-service/{src,tests} \
  services/tenant-service/{src,tests} \
  web/dashboard/{src,public,tests} \
  pipelines/event-pipeline \
  infra/{terraform,k8s,docker} \
  docs/{api,architecture/decisions,runbooks} \
  .github/workflows \
  scripts

# Step 3: Create initial commit
git add .
git commit -m "chore: initialize project structure from blueprint ${BLUEPRINT_VERSION}"
git push

# Step 4: Create Labels
echo "Creating issue labels..."

# Phase labels
gh label create "phase-1" --color "0075ca" --description "Phase 1: Core Foundation"
gh label create "phase-2" --color "0052a3" --description "Phase 2: Query Engine"
gh label create "phase-3" --color "003b87" --description "Phase 3: Multi-tenancy"
gh label create "phase-4" --color "00256b" --description "Phase 4: Production Hardening"

# Type labels
gh label create "service" --color "e4e669" --description "Service implementation"
gh label create "infrastructure" --color "d93f0b" --description "Infrastructure work"
gh label create "api-contract" --color "0e8a16" --description "API contract definition"
gh label create "security" --color "b60205" --description "Security control"
gh label create "documentation" --color "c5def5" --description "Documentation"
gh label create "adr" --color "fef2c0" --description "Architecture Decision Record"
gh label create "performance" --color "f9d0c4" --description "Performance work"

# Priority labels
gh label create "priority-critical" --color "b60205" --description "Must be in next release"
gh label create "priority-high" --color "e11d48" --description "Important for current phase"
gh label create "priority-medium" --color "f59e0b" --description "Standard priority"
gh label create "priority-low" --color "d1d5db" --description "Nice to have"

# Component labels
gh label create "comp:ingestion-service" --color "7c3aed" --description "Ingestion service component"
gh label create "comp:query-service" --color "6d28d9" --description "Query service component"
gh label create "comp:auth-service" --color "5b21b6" --description "Auth service component"
gh label create "comp:tenant-service" --color "4c1d95" --description "Tenant service component"
gh label create "comp:dashboard" --color "3730a3" --description "Dashboard frontend component"

echo "Labels created."

# Step 5: Create Milestones (from implementation.phases)
echo "Creating milestones..."
MS1=$(gh api repos/${ORG}/${REPO_NAME}/milestones \
  --method POST \
  --field title="Phase 1: Core Foundation" \
  --field description="Core ingestion pipeline, data storage, and basic API. Target: functional system ingesting events." \
  --field due_on="2026-04-25T00:00:00Z" \
  --jq '.number')

MS2=$(gh api repos/${ORG}/${REPO_NAME}/milestones \
  --method POST \
  --field title="Phase 2: Query Engine" \
  --field description="SQL query execution against ClickHouse with sub-500ms p99 latency." \
  --field due_on="2026-05-23T00:00:00Z" \
  --jq '.number')

MS3=$(gh api repos/${ORG}/${REPO_NAME}/milestones \
  --method POST \
  --field title="Phase 3: Multi-tenancy + Auth" \
  --field description="Tenant isolation, JWT authentication, billing integration." \
  --field due_on="2026-06-20T00:00:00Z" \
  --jq '.number')

MS4=$(gh api repos/${ORG}/${REPO_NAME}/milestones \
  --method POST \
  --field title="Phase 4: Production Hardening" \
  --field description="Observability, load testing, DR drills, production launch." \
  --field due_on="2026-07-18T00:00:00Z" \
  --jq '.number')

echo "Milestones created."

# Step 6: Create Issues from Deliverables
echo "Creating Phase 1 issues..."

gh issue create \
  --title "[PH-01] Ingestion Service: Event receiver endpoint" \
  --body "## Objective
Implement the HTTP endpoint that receives event payloads from clients and publishes to Kafka.

## Acceptance Criteria
- [ ] POST /ingest accepts event payload with tenant_id, event_type, properties, timestamp
- [ ] Payload validated against JSON schema with descriptive error messages on failure
- [ ] Successful events published to Kafka topic \`events.raw\`
- [ ] Returns 202 Accepted immediately (async processing)
- [ ] Unit test coverage >= 85%
- [ ] Integration test with real Kafka instance passing

## Blueprint Reference
Blueprint: Real-time Analytics API v1.0.0 | Deliverable: DEL-001" \
  --label "phase-1,service,comp:ingestion-service,priority-critical" \
  --milestone "${MS1}"

gh issue create \
  --title "[PH-01] ClickHouse: Schema design and migration system" \
  --body "## Objective
Design the ClickHouse table schema for event storage and implement the migration system.

## Acceptance Criteria
- [ ] Events table using MergeTree engine with tenant_id as partition key
- [ ] Schema supports queries by: tenant_id, event_type, time range, custom properties (JSON)
- [ ] Migration tool implemented (up/down migrations)
- [ ] Performance test: single-tenant query over 100M rows returns in < 200ms
- [ ] Schema documented in docs/architecture/

## Blueprint Reference
Blueprint: Real-time Analytics API v1.0.0 | Deliverable: DEL-002" \
  --label "phase-1,infrastructure,comp:query-service,priority-critical" \
  --milestone "${MS1}"

gh issue create \
  --title "[PH-01] ADR-001: Document ClickHouse selection decision" \
  --body "## Objective
Record the architecture decision for ClickHouse as our time-series store.

## Acceptance Criteria
- [ ] ADR written in docs/architecture/decisions/ADR-001-clickhouse.md
- [ ] Alternatives documented: TimescaleDB, Apache Druid, BigQuery
- [ ] Performance benchmarks referenced
- [ ] Accepted by tech lead

## Blueprint Reference
Blueprint: Real-time Analytics API v1.0.0 | ADR-001" \
  --label "phase-1,adr,priority-high" \
  --milestone "${MS1}"

# ... (additional issues for phases 2-4 follow the same pattern)
echo "Issues created."

# Step 7: Create GitHub Project
echo "Creating project board..."
PROJECT_ID=$(gh project create \
  --owner "${ORG}" \
  --title "Real-time Analytics API — Build Roadmap" \
  --format json | jq -r '.number')

# Add issues to project
gh project item-add "${PROJECT_ID}" --owner "${ORG}" --url "$(gh issue list --json url --jq '.[].url' | head -20)"

echo "Project board created: https://github.com/orgs/${ORG}/projects/${PROJECT_ID}"

# Step 8: Create CODEOWNERS
cat > .github/CODEOWNERS << 'EOF'
# CODEOWNERS generated from blueprint: Real-time Analytics API v1.0.0

* @tech-lead

/services/ingestion-service/ @data-platform-team
/services/query-service/     @data-platform-team
/pipelines/                  @data-platform-team
/services/auth-service/      @platform-team
/services/tenant-service/    @platform-team
/web/                        @frontend-team
/infra/                      @platform-team
/.github/                    @platform-team
/docs/                       @tech-lead
EOF

git add .github/CODEOWNERS
git commit -m "chore: add CODEOWNERS from blueprint team structure"
git push

# Step 9: Configure Branch Protection
echo "Configuring branch protection..."
gh api repos/${ORG}/${REPO_NAME}/branches/main/protection \
  --method PUT \
  --field required_status_checks='{"strict":true,"contexts":["test","security-scan","build"]}' \
  --field enforce_admins=true \
  --field required_pull_request_reviews='{"required_approving_review_count":1,"dismiss_stale_reviews":true}' \
  --field restrictions=null \
  --field required_linear_history=true

echo "Branch protection configured."

# Step 10: Summary
echo ""
echo "=== GitHub Handoff Complete ==="
echo "Repository:    https://github.com/${ORG}/${REPO_NAME}"
echo "Project Board: https://github.com/orgs/${ORG}/projects/${PROJECT_ID}"
echo "Issues:        $(gh issue list --json number --jq 'length') issues created across 4 milestones"
echo "Workflows:     CI, Deploy Staging, Deploy Production"
echo ""
echo "Next step: Clone the repository and run the Claude Code handoff to load project context."
```

---

## Handoff Acknowledgement

On completion, the adapter writes:

```json
{
  "status": "success",
  "timestamp": "2026-03-28T17:35:00Z",
  "target_system": "github",
  "artifacts_created": {
    "repository": "https://github.com/your-org/realtime-analytics-api",
    "milestones_created": 4,
    "issues_created": 23,
    "labels_created": 17,
    "workflows_created": 3,
    "project_board": "https://github.com/orgs/your-org/projects/7",
    "branch_protection": "main protected"
  },
  "blueprint_id": "a3f8c2d1-4b5e-4f6a-8c9d-0e1f2a3b4c5d",
  "blueprint_version": "1.0.0"
}
```
