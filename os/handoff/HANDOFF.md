# Handoff System Overview

> The SuperArchitect OS doesn't just design systems — it hands off complete, executable blueprints to downstream platforms for immediate action. The Handoff System is the bridge between blueprint and reality.

---

## Handoff Philosophy

**Build once. Run anywhere.**

A blueprint is not a deliverable to be filed away. It is an executable artifact — a machine-readable specification that any sufficiently capable system can transform into work. The Handoff System is the execution layer that makes this real.

The philosophy has three principles:

### 1. The Blueprint Is the Source of Truth
Every downstream artifact — GitHub issues, Linear epics, Claude project files, agent configurations — is a transformation of the blueprint, not a replacement for it. The blueprint is never modified during handoff. Transformations are one-directional. If the system design changes, the blueprint version increments first; then new handoffs are generated.

### 2. Adapters Are Dumb Transformers
Each Handoff Adapter is a pure, stateless transformer. It takes a validated blueprint and produces a target-system-specific artifact. Adapters contain no business logic, no architectural judgment, no design decisions. Those belong in the blueprint. An adapter asks only: "Given this blueprint, what does this target system need?"

### 3. Handoffs Are Auditable
Every handoff is logged: which blueprint version, which target system, what was produced, when, by whom, and what the receiving system acknowledged. If a bug is traced to a design decision, the audit trail connects it back to the exact blueprint section and the ADR that made the call.

---

## Supported Target Systems

### Claude Code (Native)
The highest-fidelity handoff. The blueprint is transformed into a `CLAUDE.md` project file that pre-loads the receiving Claude Code session with complete context: system architecture, current phase, active tasks, decision history, and custom slash commands. The receiving session starts knowing exactly what it's building and why.

**Best for**: Active development sessions, continuing a build started in SuperArchitect OS.

### OpenAI Codex / GPT-4o
Blueprint sections are transformed into structured system prompts optimized for GPT-4o's context window. Architecture components become function specifications. Implementation phases become task prompts. The handoff produces a prompt library — one prompt per implementation task.

**Best for**: Teams using OpenAI as their primary development AI.

### LangChain / LangGraph
Blueprint architecture maps directly to a LangGraph StateGraph. Components become nodes. Data flows become edges. The blueprint's team structure maps to agent roles in the graph. The handoff produces a working Python LangGraph skeleton that reflects the system's architectural topology.

**Best for**: AI-augmented systems where the architecture itself needs to run as an agent workflow.

### AutoGen
Blueprint team structure maps to an AutoGen GroupChat. Each specialist (Architect, Security Expert, Data Engineer, etc.) becomes a ConversableAgent with a system message derived from the relevant section of the blueprint. The Commander maps to the GroupChatManager. The handoff produces a ready-to-run AutoGen setup.

**Best for**: Multi-agent build automation where different AI agents own different implementation domains.

### CrewAI
Blueprint teams map to CrewAI Agents. Workflows map to CrewAI Tasks. The Commander maps to the CrewAI Crew manager. The handoff produces a `crew.py` that defines a complete CrewAI configuration for building the system.

**Best for**: Python-first teams using CrewAI for automated implementation workflows.

### GitHub
The most comprehensive project setup handoff. Creates the repository with the correct directory structure, creates all GitHub Issues from implementation tasks (with labels, milestones, and assignments), sets up the GitHub Projects board with phases as columns, creates the GitHub Actions CI/CD pipeline from the infrastructure section, and configures branch protection rules from the quality standards.

**Best for**: Teams starting a new project from scratch and wanting an immediately operational GitHub project.

### Linear
Creates a Linear project with epics from phases and stories from deliverables. Each story carries the full acceptance criteria from the blueprint's Definition of Done. Milestones map to Linear cycles. The handoff produces a fully populated Linear workspace ready for sprint planning.

**Best for**: Product-engineering teams using Linear as their primary project management tool.

### Jira
Creates a Jira project with epics from phases and stories from deliverables. Maps the team structure to Jira project roles. Creates the component hierarchy from the architecture. Produces a Jira backlog that can feed directly into sprint planning.

**Best for**: Enterprise teams with existing Jira workflows.

### Generic API
Posts the complete blueprint as a structured JSON payload to any REST endpoint. Includes the full blueprint plus a handoff envelope with metadata. Any system that can receive a JSON POST can integrate with SuperArchitect OS's output.

**Best for**: Custom internal tools, legacy project management systems, custom orchestration platforms.

---

## Handoff Router

The Handoff Router selects the correct adapter based on the blueprint's `handoff.target_systems` configuration and the available adapters.

**Decision Logic:**

```
IF target.type == "claude-code"
  → use /adapters/claude-code-handoff.md

IF target.type == "github"
  → use /adapters/github-handoff.md

IF target.type IN ["langchain", "autogen", "crewai"]
  → use /adapters/agent-framework-handoff.md
  → section selected by target.type

IF target.type IN ["linear", "jira"]
  → use /adapters/project-management-handoff.md
  → section selected by target.type

IF target.type == "generic-api"
  → serialize full blueprint to JSON
  → POST to target.endpoint with Authorization header from target.config.api_key
```

**Parallel Handoffs**: When multiple target systems are configured, adapters run in parallel. GitHub setup does not wait for Claude Code handoff to complete. All handoffs are independent.

**Priority**: The `priority` field in `handoff.target_systems` determines presentation order in the handoff summary, not execution order. All adapters run concurrently.

---

## Handoff Package

Every handoff produces a Handoff Package — a directory of artifacts for the target system.

```
handoff/{target-system-type}/
  README.md              # Human-readable summary for this target system
  {adapter-artifact}     # Primary artifact (CLAUDE.md, gh-commands.sh, crew.py, etc.)
  blueprint-ref.json     # Pointer back to the source blueprint (ID + version)
  acknowledgement.json   # Filled in after delivery
```

### Handoff Envelope

Every machine-readable handoff artifact includes a `handoff_envelope` field:

```json
{
  "handoff_envelope": {
    "blueprint_id": "a3f8c2d1-4b5e-4f6a-8c9d-0e1f2a3b4c5d",
    "blueprint_version": "1.0.0",
    "blueprint_name": "Multi-tenant SaaS Analytics Platform",
    "handoff_timestamp": "2026-03-28T17:30:00Z",
    "target_system": "github",
    "adapter_version": "1.0.0",
    "generated_by": "SuperArchitect OS v1.0"
  }
}
```

This envelope allows any receiving system to trace an artifact back to its source blueprint and version.

---

## Validation Before Handoff

No handoff proceeds without passing the full validation suite. Validation is enforced by the Commander and cannot be bypassed.

### Required Validations

**Structural Completeness**
- All required sections present and non-empty
- No `[TBD]`, `[PLACEHOLDER]`, or `[FIELD]` text in required fields
- All ID references resolve (every component ID cited in an interface exists in the component catalog)
- Blueprint status is `approved` (not `draft` or `review`)

**Content Quality**
- `problem.success_criteria` contains at least one measurable criterion
- `architecture.adrs` contains at least one ADR
- `implementation.phases` contains at least one complete phase
- `quality.definition_of_done` contains at least three items
- `security.threat_model.threats` contains at least one threat

**Schema Validation**
- All UUIDs are valid UUID v4 format
- All timestamps are valid ISO 8601 format
- All version strings are valid semver
- All enum values are from the allowed set

### Validation Report

When validation fails, the Commander produces a Validation Report listing every failed check with the field path, expected value, and actual value. Example:

```
VALIDATION FAILED — 3 issues found:

1. INCOMPLETE: problem.success_criteria
   Expected: Array with >= 1 item
   Actual: Empty array []
   Action: Strategist team must define at least one measurable success criterion

2. PLACEHOLDER: architecture.components[1].responsibility
   Expected: Non-placeholder string
   Actual: "[DESCRIPTION]"
   Action: Architecture team must complete responsibility description for COMP-002

3. MISSING_ADR: architecture.adrs
   Expected: Array with >= 1 ADR
   Actual: Empty array []
   Action: Architecture team must record at least one ADR
```

---

## Handoff Confirmation

### Automated Targets (GitHub, Linear, Jira)
The adapter captures the API response from each creation call and records it in `acknowledgement.json`:

```json
{
  "status": "success",
  "timestamp": "2026-03-28T17:32:00Z",
  "target_system": "github",
  "artifacts_created": {
    "repository": "https://github.com/org/repo",
    "issues_created": 47,
    "milestones_created": 4,
    "project_board": "https://github.com/org/repo/projects/1",
    "workflows_created": 3
  },
  "errors": []
}
```

### File-Based Targets (Claude Code, agent frameworks)
The Commander writes the artifacts to disk and produces a delivery summary. The operator confirms receipt by acknowledging the summary in the OS interface.

### API Targets (Generic API)
The HTTP response code and body are captured. 2xx is success; anything else triggers a retry with exponential backoff (3 attempts), then failure with full error detail.

---

## Partial Handoffs

Not every handoff needs to transmit the entire blueprint. Partial handoffs extract and deliver a subset of the blueprint for systems that only need part of the context.

### Architecture-Only Handoff
Transmits `meta`, `problem.statement`, `system`, `architecture`, and `api` sections. Used when an architecture review tool or diagramming system only needs structural information.

### Implementation Plan Handoff
Transmits `meta`, `implementation`, `quality`, and milestone-level `problem.success_criteria`. Used when a project management system only needs the work breakdown, not the technical specification.

### Security Handoff
Transmits `meta`, `problem.constraints`, `security`, and `quality.nfr_targets`. Used for security review tools, compliance audits, or security team handoffs.

### API Contract Handoff
Transmits `meta` and `api` in OpenAPI 3.0 format. Used to generate API documentation, mock servers, or client SDKs directly from the blueprint.

### Infrastructure Handoff
Transmits `meta`, `infrastructure`, and relevant `architecture.components` with their scaling configurations. Used to generate Terraform, Pulumi, or CloudFormation templates.

---

## Handoff Audit Log

All handoffs are recorded in the OS audit log at:
```
/home/user/superarchitect/registry/handoff-log.json
```

Each entry records:
```json
{
  "handoff_id": "UUID",
  "blueprint_id": "UUID",
  "blueprint_version": "1.0.0",
  "target_system": "github",
  "partial": false,
  "initiated_at": "ISO 8601",
  "completed_at": "ISO 8601",
  "status": "success | failure | partial",
  "artifacts": [...],
  "errors": [...]
}
```

This log is the source of truth for "what was handed off, to where, and when." It enables recovery if a handoff needs to be re-run, and provides the audit trail required for compliance and post-incident review.
