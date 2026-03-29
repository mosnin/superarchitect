# Evolution Ledger

> The system remembers how it changed, not just what it concluded.

---

## Purpose

The Evolution Ledger is the immutable history of how an architecture evolved through the kernel's iterative process. Every reroute, every score change, every mutation is recorded. This history serves three critical functions:

1. **Accountability**: Every decision can be traced to a specific audit result and rationale.
2. **Pattern detection**: Repeated failures in the same dimension reveal deeper structural problems that surface-level fixes cannot address.
3. **Learning**: The ledger is the raw material for understanding what works and what does not across builds.

Without the ledger, the system has amnesia. It makes the same mistakes, tries the same reroutes, and cannot explain why it converged on a particular design. The ledger transforms the kernel from a stateless optimizer into a learning system.

---

## Immutability Rules

1. Entries are **APPEND-ONLY**. No entry is ever modified or deleted.
2. Every reroute creates exactly one entry. No reroutes without entries. No entries without reroutes.
3. Entries must be **machine-readable** (structured format) AND **human-readable** (narrative rationale).
4. The ledger is part of the canonical system package. It ships with the architecture.
5. Tampering with the ledger (editing past entries) is a critical integrity violation.

---

## Entry Format

Each ledger entry contains exactly these fields:

```json
{
  "entry_id": "EVL-<sequence_number>",
  "iteration": 3,
  "timestamp": "2026-03-29T14:32:00Z",
  "phase_at_failure": "PHASE_6",
  "failed_dimensions": [
    {
      "dimension": "failure_awareness",
      "vector": {
        "score": 0.58,
        "evidence": 0.70,
        "uncertainty": 0.20,
        "delta": -0.05,
        "status": "fail"
      }
    }
  ],
  "passing_dimensions": [
    {
      "dimension": "coherence",
      "vector": {
        "score": 0.88,
        "evidence": 0.80,
        "uncertainty": 0.10,
        "delta": 0.03,
        "status": "pass"
      }
    }
  ],
  "reroute_target": "PHASE_5",
  "reroute_reason": "Failure awareness below threshold. Missing containment strategies for 3 of 5 identified failure modes. Cascading failure path from payment service to order service is uncontained.",
  "mutation_applied": "Added circuit breaker between payment and order services. Added dead letter queue for failed payment events. Defined timeout and retry policy for all external service calls.",
  "mutation_scope": "Structural — added 2 new components (circuit breaker, DLQ) and 3 new policies (timeout, retry, fallback).",
  "result": "IMPROVED",
  "post_mutation_scores": {
    "failure_awareness": {
      "score": 0.81,
      "evidence": 0.75,
      "uncertainty": 0.15,
      "delta": 0.23,
      "status": "pass"
    }
  }
}
```

### Field Definitions

| Field | Type | Description |
|---|---|---|
| `entry_id` | string | Unique identifier. Format: `EVL-<N>` where N is sequential. |
| `iteration` | integer | Which iteration of the kernel loop this entry belongs to. |
| `timestamp` | ISO 8601 | When the entry was created. |
| `phase_at_failure` | string | Which phase was executing when the audit failure occurred. |
| `failed_dimensions` | array | All dimensions that failed audit, with their full confidence vectors. |
| `passing_dimensions` | array | All dimensions that passed, for complete record. |
| `reroute_target` | string | The phase the kernel rerouted to. |
| `reroute_reason` | string | Human-readable explanation of why this reroute target was chosen. |
| `mutation_applied` | string | What changed in the architecture as a result of the reroute. |
| `mutation_scope` | string | Categorization of the change: structural, behavioral, interface, documentation. |
| `result` | enum | `IMPROVED`, `UNCHANGED`, `REGRESSED` — based on post-mutation scores vs pre-mutation. |
| `post_mutation_scores` | object | Confidence vectors for the previously-failed dimensions after the mutation was applied and re-audited. |

---

## How to Read the Ledger

### Reading the Evolution Story

The ledger tells a story. Read it chronologically:

1. **Entry 1**: What was the first failure? This reveals the initial design's weakest point.
2. **Entries 2-N**: How did the system respond? Each mutation should address the specific weakness identified.
3. **Results**: Did mutations work? `IMPROVED` means the reroute was effective. `UNCHANGED` means the reroute target was wrong. `REGRESSED` means the mutation introduced new problems.
4. **Final state**: The last entry's post_mutation_scores represent the system's final quality profile.

### Key Questions the Ledger Answers

- How many iterations did convergence require?
- Which dimensions were hardest to satisfy?
- Were any reroutes ineffective (result = UNCHANGED)?
- Did any mutations cause regressions?
- Is there a pattern of the same dimension failing repeatedly?

---

## Pattern Detection

The ledger enables detection of systemic problems that individual audit cycles cannot see:

### Recurring Dimension Failure
**Pattern**: The same dimension fails in 3+ consecutive entries.
**Diagnosis**: Surface-level fixes are not addressing the root cause. The problem is structural.
**Response**: Escalate to a deeper reroute (earlier phase) or trigger overdrive mode for that dimension.

### Oscillation
**Pattern**: A dimension alternates between pass and fail across entries.
**Diagnosis**: Mutations that fix one thing break another. Components are too tightly coupled.
**Response**: Reroute to Phase 5 (Structural Synthesis) to improve isolation between the affected components.

### Cascade Regression
**Pattern**: Fixing dimension A causes dimension B to regress.
**Diagnosis**: The dimensions are in tension (see cross-dimension interactions in `universal_backbone.md`).
**Response**: Document the trade-off explicitly. Find a balanced solution that satisfies both, even if neither reaches maximum score.

### Stagnation
**Pattern**: Delta near zero for 3+ iterations across all dimensions.
**Diagnosis**: The system has reached a local optimum that does not meet thresholds.
**Response**: Trigger overdrive mode or escalate to human operator for guidance on which dimension to prioritize.

---

## Integration with Canonical System Package

The evolution ledger is embedded in the canonical system package as the `evolution` array:

```json
{
  "system_package": {
    "identity": { ... },
    "architecture": { ... },
    "audit_results": { ... },
    "evolution": [
      { "entry_id": "EVL-1", ... },
      { "entry_id": "EVL-2", ... },
      { "entry_id": "EVL-3", ... }
    ]
  }
}
```

The evolution array is a first-class part of the deliverable. It is not debug output — it is architectural documentation that explains WHY the architecture looks the way it does.

---

## Example: 3-Iteration Evolution

**Iteration 1** — Initial audit after Phase 5 (Structural Synthesis):
```
EVL-1: failure_awareness scored 0.45 (fail). No failure modes identified for external service calls.
       Reroute → Phase 5. Mutation: Add failure mode analysis for all integration points.
       Result: IMPROVED. Post-mutation score: 0.63 (still fail, but improved).
```

**Iteration 2** — Second audit after rerouted Phase 5:
```
EVL-2: failure_awareness scored 0.63 (fail). Failure modes identified but containment missing.
       Reroute → Phase 5. Mutation: Add circuit breakers, timeouts, and fallback strategies.
       Result: IMPROVED. Post-mutation score: 0.81 (pass).
```

**Iteration 3** — Third audit:
```
EVL-3: All dimensions pass. No reroute needed.
       System advances to Phase 7 (Packaging).
```

This 3-iteration evolution is normal and healthy. The system identified a weakness, iteratively improved it, and converged. The ledger documents exactly how and why.

---

*Evolution Ledger v1.0 — Kernel Audit Infrastructure*
