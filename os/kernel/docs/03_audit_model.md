# Audit Model

> The audit layer is the kernel's measurable optimization engine.
> It transforms subjective quality judgments into dimensional scores, evidence-backed routing decisions, and a permanent record of every iteration.

---

## Purpose

The audit model exists to answer one question at the end of every synthesis pass:

> **Is this design good enough to build, and if not, what specifically must improve?**

"Good enough" is not a feeling. It is a vector of dimensional scores, each compared against a threshold defined in the success model. The audit layer produces this vector, interprets it, and either advances the pipeline to packaging or reroutes to the shallowest phase that can address the deficiency.

---

## Audit Components

Every audit pass produces an audit record with the following components:

### Score (per dimension)
A value from 0.0 to 1.0 representing how well the synthesized design satisfies a given success dimension. Scores are not arbitrary — each is accompanied by evidence.

### Evidence Strength
A qualifier on each score: `strong`, `moderate`, or `weak`. Strong evidence means the score is derived from concrete structural analysis (e.g., "the design includes circuit breakers on all external calls" supports a high failure_awareness score). Weak evidence means the score is inferred from general properties.

### Uncertainty
A value from 0.0 to 1.0 representing how confident the audit layer is in its score. High uncertainty (>0.5) triggers mandatory documentation of what additional information would reduce it.

### Delta (for re-audits)
On corrective iterations, the delta records the change in score from the previous audit pass. A negative delta on the failing dimension is a critical signal — the correction made things worse.

### Failure Explanation
When a dimension scores below threshold, the audit layer produces a structured explanation: what the design lacks, what impact this has, and what a passing design would include.

### Reroute Target
The kernel phase to which the pipeline should return. Determined by the routing rule (shallowest phase that can fix the defect).

### Suggested Mutation
A concrete description of what should change in the reroute target phase. Not a vague directive ("improve security") but a specific action ("add rate limiting on the public API gateway and implement token bucket algorithm with per-tenant quotas").

---

## The Universal Backbone (8 Dimensions)

Every system, regardless of domain, is scored on these 8 dimensions:

| Dimension | What It Measures | Typical Threshold |
|-----------|-----------------|-------------------|
| **correctness** | Does the design satisfy the stated intent and constraints? | 0.8 |
| **modularity** | Are subsystems well-bounded with clean interfaces? | 0.7 |
| **scalability** | Can the system handle growth in load, data, and users? | 0.7 |
| **security** | Does the design protect against threats and enforce access control? | 0.75 |
| **operability** | Can the system be deployed, monitored, and maintained? | 0.7 |
| **failure_awareness** | Does the design anticipate and contain failures? | 0.7 |
| **implementability** | Is the design concrete enough for engineers to build from? | 0.75 |
| **coherence** | Are all parts of the design internally consistent? | 0.8 |

These thresholds are defaults. The success model may adjust them based on consequence level:
- **Low consequence**: All thresholds reduced by 0.1
- **High consequence**: security and failure_awareness thresholds increased by 0.05
- **Critical consequence**: All thresholds increased by 0.1

---

## Project-Derived Dimensions

Beyond the universal backbone, the success model defines project-specific dimensions derived from the intent. Examples:

| Project Type | Example Dimensions |
|-------------|-------------------|
| Real-time collaboration | `latency`, `concurrency`, `collaboration_fidelity`, `conflict_resolution` |
| Financial platform | `transaction_integrity`, `audit_trail`, `regulatory_compliance` |
| Data pipeline | `throughput`, `data_quality`, `lineage_completeness`, `recovery_time` |
| Consumer mobile app | `offline_capability`, `startup_time`, `battery_efficiency` |

Project-derived dimensions are scored with the same rigor as backbone dimensions. They have thresholds, evidence requirements, and failure explanations.

---

## The Confidence Vector Model

The full audit output is a **confidence vector** — a multi-dimensional representation of design quality:

```yaml
audit_vector:
  dimensions:
    correctness:      { score: 0.88, evidence: strong, uncertainty: 0.1 }
    modularity:       { score: 0.82, evidence: strong, uncertainty: 0.1 }
    scalability:      { score: 0.79, evidence: moderate, uncertainty: 0.2 }
    security:         { score: 0.81, evidence: strong, uncertainty: 0.15 }
    operability:      { score: 0.75, evidence: moderate, uncertainty: 0.2 }
    failure_awareness: { score: 0.62, evidence: weak, uncertainty: 0.3 }  # BELOW THRESHOLD
    implementability: { score: 0.85, evidence: strong, uncertainty: 0.1 }
    coherence:        { score: 0.90, evidence: strong, uncertainty: 0.05 }
    # project-specific
    latency:          { score: 0.83, evidence: moderate, uncertainty: 0.15 }
    concurrency:      { score: 0.78, evidence: moderate, uncertainty: 0.2 }
  overall_pass: false
  failing_dimensions: ["failure_awareness"]
  weakest_dimension: "failure_awareness"
```

The vector model enables:
- **Precise diagnosis**: The kernel knows exactly which dimension is weak, not just that "something is wrong"
- **Targeted correction**: The suggested mutation addresses the specific failing dimension
- **Progress tracking**: Successive audit passes show whether corrections are working
- **Tradeoff visibility**: When one dimension improves, engineers can see if others regressed

---

## Routing: How Audit Results Drive Rerouting

When the audit identifies a failing dimension, the routing logic determines the reroute target:

```
IF failing_dimension relates to intent interpretation:
    reroute → Phase 1 (Intent Compilation)

ELIF failing_dimension relates to success criteria:
    reroute → Phase 2 (Success Model)

ELIF failing_dimension indicates missing architectural approach:
    reroute → Phase 3 (Architecture Search)

ELIF failing_dimension indicates wrong candidate selection:
    reroute → Phase 4 (Comparative Reasoning)

ELSE (most common):
    reroute → Phase 5 (Structural Synthesis)
```

In practice, 80% of reroutes target Phase 5. The design direction is correct, but the synthesis is incomplete — a missing circuit breaker, an underspecified interface, or an unanalyzed failure mode.

### Reroute Constraints

- **Maximum iterations**: 3 corrective iterations per build (configurable by consequence level)
- **Monotonic improvement required**: Each re-audit must show improvement on the failing dimension. If the score decreases or stagnates, the kernel escalates to the human operator.
- **No skip-ahead**: After rerouting, the pipeline must re-execute all phases from the reroute target forward. You cannot reroute to Phase 3 and skip directly to Phase 6.

---

## The Evolution Ledger

Every reroute is permanently recorded in the evolution ledger:

```yaml
evolution_ledger:
  - iteration: 1
    timestamp: "2026-03-29T01:30:00Z"
    audit_pass: 1
    failed_dimensions: ["failure_awareness"]
    scores_at_failure:
      failure_awareness: 0.62
    reroute_target: "phase_5_structural_synthesis"
    mutation_applied: >
      Added circuit breakers on all external service calls.
      Added offline mode with local-first CRDT sync.
      Added graceful degradation tiers: full → reduced → read-only → offline.
      Added health check cascade with dependency-aware status.
    outcome: "re-audit passed with failure_awareness: 0.81"
```

The ledger serves multiple purposes:
- **Audit trail**: Anyone reviewing the package can see what was tried and why
- **Learning signal**: Patterns in reroutes reveal systematic weaknesses in the kernel's synthesis
- **Estimation**: The number and depth of reroutes informs time estimates for similar future builds

---

## How Audit Connects to OS Teams

The audit phase dispatches evaluation work to specialist teams:

| Audit Dimension | Evaluating Team | Method |
|----------------|----------------|--------|
| correctness | Team 2 (Architecture) | Intent-to-design traceability review |
| modularity | Team 2 (Architecture) | Coupling analysis, boundary review |
| scalability | Team 2 (Architecture) + Team 6 (DevOps) | Load modeling, bottleneck analysis |
| security | Team 7 (Security) | Threat model review, STRIDE analysis |
| operability | Team 6 (DevOps) | Deployment review, runbook evaluation |
| failure_awareness | Team 8 (QA) | Failure mode analysis, chaos scenario review |
| implementability | Team 3 (Backend) + Team 4 (Frontend) | "Can we build this?" review |
| coherence | Team 2 (Architecture) | Cross-section consistency check |
| Project-specific | Varies by dimension | Domain-appropriate evaluation |

The Commander dispatches these evaluations in parallel where possible. Each team returns a structured assessment that the audit layer aggregates into the confidence vector.

---

## Audit Quality Standards

The audit layer itself must meet quality standards:

1. **No hand-waving**: Every score must cite specific structural evidence from the design
2. **No grade inflation**: Scores must reflect actual design quality, not aspiration
3. **No false passes**: A dimension that lacks evidence defaults to a low score, not a passing one
4. **Uncertainty honesty**: When the audit layer cannot fully evaluate a dimension, uncertainty must be flagged high
5. **Actionable failures**: Every failure explanation must include enough detail for the target phase to act on it without guessing

---

*Audit Model v1.0 — SuperArchitect OS*
