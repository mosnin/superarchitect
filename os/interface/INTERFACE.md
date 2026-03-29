# One-Command Interface — SuperArchitect OS

**Version:** 1.0
**Owner:** Foundation Team (Team 1)
**Last Updated:** 2026-03-29

---

## Purpose

The Interface Layer is the human-facing surface of the SuperArchitect OS. It translates natural language requests into structured OS operations. The goal is zero-friction interaction: one command triggers an entire autonomous workflow.

---

## 1. Command Reference

### Primary Commands

| Command | Purpose | Complexity |
|---------|---------|------------|
| `/build-system <description>` | Build a complete system from scratch | Full workflow |
| `/audit-system [path]` | Audit an existing system | Full audit workflow |
| `/spawn-team <team> <task>` | Activate a single specialist team | Single team |
| `/build-feature <description>` | Add a feature to existing system | Feature workflow |
| `/refactor <scope>` | Refactor existing code | Refactor workflow |
| `/research <topic>` | Research sprint on a topic | Research workflow |

### Operational Commands

| Command | Purpose |
|---------|---------|
| `/status` | Report current OS state, active tasks, phase progress |
| `/resume` | Resume an interrupted build from last checkpoint |
| `/escalate <question>` | Surface a decision to the human operator |
| `/extend-os <type> <name>` | Add a new team, workflow, or pattern to the OS |

---

## 2. Natural Language Understanding

The Interface Layer parses natural language system descriptions and extracts structured intent.

### Example Inputs and Interpretations

**Input:** "Build a SaaS platform for managing restaurant reservations"
```yaml
intent: build-system
domain: hospitality / restaurant management
system_type: SaaS
core_features: [reservation management, availability tracking]
implied_requirements: [multi-tenant, user auth, notifications, real-time updates]
```

**Input:** "I need an API for processing payments with Stripe integration"
```yaml
intent: build-system
domain: fintech / payments
system_type: API
core_features: [payment processing, Stripe integration]
implied_requirements: [PCI compliance, idempotency, webhook handling, audit trail]
```

**Input:** "Audit the codebase in ./backend for security issues"
```yaml
intent: audit-system
scope: ./backend
focus: security
implied_actions: [dependency scan, auth review, input validation check, secrets scan]
```

**Input:** "Add user notifications via email and push"
```yaml
intent: build-feature
feature: notification system
channels: [email, push]
implied_requirements: [notification preferences, templates, delivery tracking, retry]
```

### Ambiguity Resolution

When the description is ambiguous, the OS:
1. Makes reasonable assumptions based on domain knowledge
2. Documents all assumptions explicitly
3. Proceeds with the most common interpretation
4. Lists assumptions in the delivery summary for human review

**The OS never blocks on ambiguity for technical decisions.** It only escalates for business decisions where the wrong choice has irreversible consequences.

---

## 3. Progress Reporting

### Phase Progress Display

During a build, the OS reports progress:

```
[SuperArchitect OS] Build in progress...

Phase 1: Discovery .............. [COMPLETE]
  - System type: SaaS Platform
  - Complexity: Medium
  - 12 functional requirements identified
  - 8 non-functional requirements defined

Phase 2: Architecture ........... [COMPLETE]
  - Pattern: Modular Monolith
  - Stack: TypeScript, Fastify, PostgreSQL, Redis
  - 5 modules identified
  - 3 ADRs produced

Phase 3: Security ............... [COMPLETE]
  - Threat model: 8 threats identified, all mitigated
  - Auth: JWT + refresh tokens via Auth0
  - Encryption: TLS 1.3 + AES-256 at rest

Phase 4: Implementation ......... [IN PROGRESS]
  - Backend: 60% complete (3/5 modules)
  - Infrastructure: 80% complete
  - Tests: 45 passing

Phase 5: Quality Assurance ...... [PENDING]
Phase 6: Documentation .......... [PENDING]
Phase 7: Delivery ............... [PENDING]
```

### Quality Gate Reports

When a quality gate is evaluated:

```
[Quality Gate: Phase 4 - Implementation]
  [PASS] All unit tests pass (45/45)
  [PASS] Code coverage: 92% (threshold: 90%)
  [PASS] Linting: 0 errors, 0 warnings
  [PASS] Security scan: 0 critical, 0 high
  [PASS] API contracts match specification
  [PASS] Infrastructure deploys successfully
  RESULT: PASS - Proceeding to Phase 5
```

---

## 4. Delivery Summary Format

When a build completes, the OS presents a structured delivery summary:

```markdown
# Build Complete: [System Name]

## System Overview
[2-3 sentence description of what was built]

## Architecture
- Pattern: [selected pattern]
- Services/Modules: [list]
- Technology Stack: [language, framework, database, etc.]

## What Was Built
- [Module 1]: [description]
- [Module 2]: [description]
- ...

## Key Technical Decisions
1. [Decision]: [rationale]
2. [Decision]: [rationale]

## Security Posture
- Authentication: [mechanism]
- Authorization: [model]
- Data encryption: [approach]

## Quality Summary
- Test count: [number]
- Coverage: [percentage]
- Security scan: [result]

## How to Run
[Setup and run instructions]

## Known Limitations
- [Limitation 1]
- [Limitation 2]

## Recommended Next Steps
1. [Next step]
2. [Next step]

## Assumptions Made
- [Assumption 1]
- [Assumption 2]
```

---

## 5. Error and Escalation Display

### Error Reporting

When the OS encounters an error it can resolve:
```
[WARNING] Database migration conflict detected in orders module.
  Action taken: Resolved by reordering migrations (20260329_001 before 20260329_002).
  Impact: None. Build continues.
```

### Escalation Reporting

When the OS needs human input:
```
[ESCALATION] Business Decision Required

Context: The system requires user authentication. Two viable options exist:

Option A: Self-hosted (Keycloak)
  - Cost: $0 license, ~$200/month infrastructure
  - Control: Full control, self-managed
  - Risk: Operational burden, security responsibility

Option B: Managed (Auth0)
  - Cost: ~$500/month at expected scale
  - Control: Limited customization
  - Risk: Vendor dependency

Recommendation: Auth0 (lower operational risk for the team size)

Please choose: [A] Self-hosted  [B] Managed  [C] Other (specify)
```

---

## 6. Interaction Modes

### Autonomous Mode (Default)
The OS makes all decisions and reports results. Human interaction is minimal.
- Best for: Experienced teams who trust the OS, standard system types.
- Human input: Only at escalation points and final delivery.

### Guided Mode
The OS pauses at phase boundaries for human review and approval.
- Best for: First-time users, high-stakes systems, learning the OS.
- Human input: After each phase quality gate.

### Collaborative Mode
The OS works alongside the human, accepting corrections and overrides at any point.
- Best for: Systems with unusual requirements, domain experts with specific preferences.
- Human input: As much as the human wants to provide.

### Mode Selection
```
/build-system --mode=autonomous <description>  (default)
/build-system --mode=guided <description>
/build-system --mode=collaborative <description>
```

---

## 7. Session Management

### Checkpoint System
The OS saves progress at every phase boundary:
```
.superarchitect/
  session/
    current.json         # Current session state
    checkpoints/
      phase-1-complete.json
      phase-2-complete.json
      ...
```

### Resume Capability
If a session is interrupted:
```
/resume
```
The OS loads the last checkpoint and continues from where it left off. All completed phases are preserved.

### Session History
```
/status
```
Shows current session state, completed phases, active tasks, and next steps.

---

*The Interface Layer makes the OS accessible. Complex orchestration happens behind the scenes. The human sees a single command go in and a complete system come out.*
