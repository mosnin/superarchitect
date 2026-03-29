# The Canonical System Package

> One YAML object per project. Single source of truth.
> Every kernel phase reads from and writes to this package.

---

## Definition

The canonical system package is a structured YAML document that contains the complete architectural description of a software system. It is the **sole output** of the kernel pipeline. All other artifacts — blueprints, handoff manifests, audit reports — are derived from it.

A canonical system package is:

- **Complete**: All 11 system primitives are populated with implementation-ready detail.
- **Self-contained**: The package includes its own success model, audit history, and decision rationale. No external documents are needed to understand the design.
- **Versioned**: Every mutation to the package is recorded in the evolution ledger with timestamp, triggering phase, and rationale.
- **Machine-readable**: The YAML structure follows a strict schema, enabling automated validation, diffing, and transformation.
- **Human-legible**: Despite being structured data, the package is written in clear technical language that any senior engineer can read and evaluate.

## Why One Package Matters

### Prevents File Soup

Without a canonical package, architectural decisions scatter across multiple documents: an architecture diagram here, an API spec there, a security review somewhere else. These documents drift apart. The canonical package keeps everything in one place, in one format, with one version history.

### Enables Traceability

Every element in the package can be traced back to the intent that motivated it, the success dimension it serves, and the candidate architecture it originated from. This traceability is not aspirational — it is structural. The YAML schema enforces it.

### Enables Replayability

Given the same intent input, the kernel should produce an equivalent (not necessarily identical) package. Because every decision is recorded with rationale, an engineer can replay the reasoning and verify that the architecture makes sense — even months after the build.

### Enables Diffing

When requirements change, the kernel can re-run with modified intent and produce a new package. Diffing the old and new packages reveals exactly what changed and why — at the level of individual subsystems, interfaces, and flows.

---

## The 11 Required Sections

Every canonical system package contains exactly these sections:

### 1. `metadata`
Project identification, timestamps, kernel version, consequence level, and build configuration.

### 2. `intent`
The compiled intent object from Phase 1. Contains raw request, parsed objective, desired outcomes, known constraints, consequence level, and ambiguity notes.

### 3. `success_model`
The full success model from Phase 2. Contains backbone dimensions, project-specific dimensions, tradeoff priority, and thresholds. This is the rubric the entire design is measured against.

### 4. `subsystems`
The major functional components of the system. Each subsystem has a name, responsibility description, technology recommendations, scaling characteristics, and failure modes.

### 5. `interfaces`
The contracts between subsystems. Each interface specifies the communicating parties, protocol (REST, gRPC, WebSocket, message queue, etc.), payload schema, error contract, and SLA.

### 6. `data_model`
The core entities, their relationships, ownership (which subsystem owns each entity), storage technology, and consistency requirements.

### 7. `flows`
End-to-end paths through the system for key operations. Each flow names the trigger, the sequence of subsystem interactions, the happy path, the error paths, and the latency budget.

### 8. `dependencies`
External systems, third-party services, and libraries the system relies on. Each dependency includes purpose, criticality (can the system function without it?), fallback strategy, and version constraints.

### 9. `control_points`
Gates, limiters, and checkpoints that govern system behavior. Rate limiters, circuit breakers, feature flags, approval gates, and access control checkpoints. Each control point specifies its trigger condition, action, and recovery behavior.

### 10. `feedback_loops`
Monitoring, alerting, and self-regulation mechanisms. Each feedback loop specifies what is measured, the thresholds, the alert channels, and the automated response (if any).

### 11. `evolution_paths`
How the system is designed to change over time. Near-term evolution (next 3 months), medium-term (3-12 months), and long-term (1-3 years). Each evolution path identifies the trigger condition, the required changes, and the migration strategy.

---

## Additional Package Sections

Beyond the 11 system primitives, the package includes:

### `audit_history`
Complete record of every audit pass. Each entry contains the dimensional score vector, the pass/fail decision, the failing dimensions (if any), and the reroute target.

### `evolution_ledger`
Record of every corrective iteration. Each entry contains the iteration number, the triggering audit result, the reroute target phase, the mutation applied, and the outcome.

### `decision_log`
Significant architectural decisions with rationale. Each entry follows the ADR (Architecture Decision Record) format: context, decision, rationale, consequences, and status.

### `output_artifacts`
Registry of all derived artifacts generated from this package, with checksums and generation timestamps.

---

## Behavioral Rule

The canonical package is not a write-once artifact. It is a **living document** that grows through the kernel pipeline:

> **All phases read from and append to the package. No phase overwrites another phase's output without recording the change in the evolution ledger.**

This rule ensures that:
- Phase 3 (Architecture Search) can reference Phase 1 (Intent) constraints directly
- Phase 5 (Structural Synthesis) builds on Phase 4's (Comparative Reasoning) selection
- Phase 6 (Audit) can trace any design element back to the intent that motivated it
- Corrective iterations preserve the history of what was tried and why

---

## Package Lifecycle

```
Phase 1  ──>  Create package shell + populate intent section
Phase 2  ──>  Populate success_model section
Phase 3  ──>  Populate candidates section (temporary; pruned after Phase 4)
Phase 4  ──>  Populate comparative_matrix + winner selection
Phase 5  ──>  Populate all 11 system primitive sections
Phase 6  ──>  Populate audit_history + evolution_ledger (if rerouting)
           ──>  On reroute: return to target phase, mutate, re-audit
Phase 7  ──>  Finalize package + generate derived artifacts
           ──>  Persist to output directory
```

The candidates section is marked as `_working` data after Phase 4. It remains in the package for traceability but is not included in derived artifacts.

---

## Relationship to the Blueprint Protocol

The **system_blueprint.md** file is the human-readable architecture document that engineers use during implementation. It is a common misconception that the blueprint is an independent document that architects write freeform.

In the kernel model:

> **The blueprint is DERIVED from the package. It is not written independently.**

The generation flow is:

```
final_system_package.yaml  ──>  blueprint generator  ──>  system_blueprint.md
```

The blueprint generator reads the canonical package and produces a structured Markdown document with:
- Executive summary (from intent + success model)
- Architecture overview (from subsystems + interfaces)
- Data model documentation (from data_model section)
- Key flows with sequence descriptions (from flows section)
- Failure analysis (from control_points + feedback_loops)
- Evolution roadmap (from evolution_paths)
- Appendices: decision log, audit history, dependency inventory

If the package changes, the blueprint is regenerated. The blueprint never contains information that is not in the package.

---

## Schema Validation

The canonical package must validate against the schema defined in `os/kernel/schemas/`. Validation checks:

- All 11 required sections present
- All required fields within each section populated
- All interface references resolve to defined subsystems
- All flow steps reference defined subsystems and interfaces
- All evolution ledger entries have valid phase references
- Consequence level is one of: low, medium, high, critical
- All scores are in range 0.0-1.0
- All thresholds are in range 0.0-1.0

Schema validation runs automatically at Phase 7. It may also be invoked manually at any phase for early error detection.

---

*Canonical System Package v1.0 — Universal Systems Kernel*
