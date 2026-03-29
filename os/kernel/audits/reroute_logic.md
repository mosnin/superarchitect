# Reroute Logic Engine

> Route back to the SHALLOWEST stage that can fix the defect.

---

## Core Principle

When an audit dimension fails, the system must reroute to an earlier phase for correction. The reroute target is always the **shallowest** (earliest) phase that has the authority and capability to fix the identified deficiency.

**Why shallow?**
1. **Preserves work**: Deeper phases have already produced outputs that may be valid. Rerouting to a shallow phase preserves the maximum amount of completed work.
2. **Minimizes waste**: Each phase consumes compute, time, and context. Unnecessary re-execution of deep phases wastes all three.
3. **Fixes root causes**: Shallow reroutes address the source of the problem. Deep reroutes often only treat symptoms, leading to repeated failures.

**The exception**: When a shallow reroute has already been tried and failed (delta < 0.05), escalate to a deeper reroute. The evolution ledger tracks this.

---

## Reroute Mapping Table

This table maps specific audit findings to their correct reroute targets. The Controller Loop uses this table to determine where to send the system when an audit fails.

### Phase 1 — Intent Compilation (Reroute Here When...)

| Symptom | Evidence | Reroute Rationale |
|---|---|---|
| Objective is unclear or contradictory | Coherence < 0.5 with notes indicating purpose confusion | The system does not know what it is trying to do. No amount of architectural work fixes this. |
| Success criteria cannot be defined | Completeness fail on "Purpose" primitive | If you cannot define success, you cannot build toward it. |
| Stakeholder needs are contradictory | Coherence fail with conflicting requirement traces | Conflicting inputs must be resolved before design begins. |
| Stated objective is fundamentally infeasible | Implementability < 0.3 with technology/physics constraints | The goal itself must change. |

### Phase 2 — Success Model (Reroute Here When...)

| Symptom | Evidence | Reroute Rationale |
|---|---|---|
| Success criteria too vague to measure | Completeness fail on success metrics | Cannot audit what you cannot measure. |
| Thresholds are arbitrary or missing | Audit unable to determine pass/fail boundaries | Domain-specific thresholds must be defined before architecture can be evaluated. |
| Trade-off priorities are unclear | Multiple dimensions in tension with no documented priority | The success model must specify which qualities matter most. |

### Phase 3 — Architecture Search (Reroute Here When...)

| Symptom | Evidence | Reroute Rationale |
|---|---|---|
| All candidates are too similar | Adaptability low because alternatives were not explored | Insufficient design space exploration. Need fundamentally different approaches. |
| Poor candidate diversity | Efficiency or Adaptability fail because the chosen architecture is the wrong paradigm | The right architecture was never considered. |
| Chosen architecture is fundamentally infeasible | Implementability < 0.5 on technology or skill grounds | Need a different architectural approach entirely. |
| Architecture cannot support required failure modes | Failure Awareness < 0.5 with structural limitations | The architectural style lacks the primitives needed for resilience. |

### Phase 4 — Comparative Reasoning (Reroute Here When...)

| Symptom | Evidence | Reroute Rationale |
|---|---|---|
| Weak ranking logic | Internal Consistency fail in candidate comparison | Candidates were compared on wrong criteria or with flawed logic. |
| Narrow score gap + high uncertainty | Top candidates within 0.1 AND uncertainty > 0.4 | Cannot confidently select a winner. Need deeper comparison or new candidates. |
| Winner does not satisfy success criteria | Completeness fail relative to success model | The comparison was valid but none of the candidates are good enough. Reroute to Phase 3 for new candidates. |

### Phase 5 — Structural Synthesis (Reroute Here When...)

| Symptom | Evidence | Reroute Rationale |
|---|---|---|
| Weak subsystem boundaries | Coherence < 0.7 with coupling issues | Boundaries need to be redrawn. |
| Interface mismatches | Internal Consistency fail on contract matching | Interfaces need to be reconciled. |
| Missing feedback loops | Completeness fail on "Feedback" primitive | Monitoring and adjustment mechanisms must be designed. |
| Missing failure containment | Failure Awareness < 0.7 with uncontained failure paths | Circuit breakers, bulkheads, and recovery strategies must be added. |
| Unnecessary complexity | Efficiency < 0.7 with removable components identified | Structure needs simplification. |
| Tight coupling | Adaptability < 0.7 with high change impact scores | Components need better isolation. |
| Illegible structure | Legibility < 0.7 with tangled flows | Flows and boundaries need clarification. |

### Phase 6 — Audit (Reroute Here When...)

| Symptom | Evidence | Reroute Rationale |
|---|---|---|
| Weak evidence scores across dimensions | Multiple dimensions with evidence < 0.4 | The audit itself was insufficient. Re-audit with deeper analysis. |
| Confidence theater detected | High scores + low evidence + high uncertainty | Assessment is unreliable. Need genuine analysis, not surface checks. |

### Phase 7 — Packaging (Reroute Here When...)

| Symptom | Evidence | Reroute Rationale |
|---|---|---|
| Documentation gaps | Legibility < 0.7 with missing docs, but structure is clear | The architecture is sound but not well-documented. |
| Missing evolution documentation | Adaptability warn due to undocumented evolution paths | Evolution paths exist in the architecture but are not captured in the package. |

---

## Reroute Decision Algorithm

```
FUNCTION determine_reroute_target(failed_dimensions, evolution_ledger):

  FOR each failed_dimension IN failed_dimensions:
    candidate_phase = lookup_mapping_table(failed_dimension)

    # Check if we already tried this reroute
    previous_reroutes = ledger.filter(
      dimension = failed_dimension.name,
      target = candidate_phase
    )

    IF previous_reroutes.count >= 2 AND last_delta < 0.05:
      # This reroute target is not working. Go deeper.
      candidate_phase = go_one_phase_earlier(candidate_phase)

    IF previous_reroutes.count >= 3:
      # Three attempts at the same dimension. Trigger overdrive.
      TRIGGER overdrive_mode(failed_dimension)

    reroute_targets.add(candidate_phase)

  # Route to the SHALLOWEST (earliest) phase among all targets
  RETURN min(reroute_targets)
```

**Key rule**: When multiple dimensions fail simultaneously, reroute to the shallowest target among all failing dimensions. This is because earlier phases constrain later phases — fixing an earlier phase often resolves downstream failures automatically.

---

## Safety Valves

### Max Reroute Iterations: 5

After 5 reroute iterations without all dimensions passing, the kernel does NOT continue indefinitely. It:

1. Compiles the current best state of the architecture.
2. Documents all failing dimensions and their histories in the evolution ledger.
3. Emits a structured escalation to the human operator with:
   - What dimensions are failing and why.
   - What reroutes have been tried and their results.
   - Recommended actions (with trade-offs).

### Overdrive Trigger

**Condition**: Any dimension falls below 0.5 after 2 reroute attempts.

**Response**: Overdrive mode is activated for that specific dimension. Overdrive spawns a focused specialist pass that:
- Performs deep-dive analysis on ONLY the failing dimension.
- Generates 3 targeted mutations (not full re-architecture).
- Selects the best mutation based on predicted improvement.
- Applies the mutation and re-audits.

Overdrive consumes significantly more resources but is the last resort before escalation.

### Escalation Trigger

**Condition**: 3 consecutive iterations where delta < 0.05 for any failing dimension.

**Response**: The system is not converging. Possible causes:
- The objective is inherently contradictory (reroute to Phase 1 will not help).
- The trade-off requires human judgment (business decision).
- The problem is outside the system's capability (novel domain).

Escalation follows the protocol in `runtime/escalation_policy.md`.

---

## Reroute Impact Preservation

When rerouting to an earlier phase, not all later-phase work is discarded:

| Reroute To | Preserved | Discarded |
|---|---|---|
| Phase 1 | Nothing | Everything (full restart) |
| Phase 2 | Phase 1 outputs | Phases 2-6 |
| Phase 3 | Phases 1-2 outputs | Phases 3-6 |
| Phase 4 | Phases 1-3 outputs (candidates regenerated with constraints) | Phases 4-6 |
| Phase 5 | Phases 1-4 outputs, selected candidate | Phases 5-6 |
| Phase 6 | Phases 1-5 outputs (re-audit only) | Phase 6 only |
| Phase 7 | Phases 1-6 outputs (repackage only) | Phase 7 only |

**Critical rule**: When rerouting to Phase N, the system carries forward all context from phases 1 through N-1, plus the audit feedback that triggered the reroute. The rerouted phase receives explicit instructions about what to fix, derived from the failed dimension's confidence vector and the evolution ledger.

---

## Reroute Message Format

When the Controller Loop dispatches a reroute, it sends this structured message:

```json
{
  "type": "REROUTE",
  "target_phase": "PHASE_5",
  "iteration": 3,
  "trigger": {
    "failed_dimensions": ["failure_awareness", "adaptability"],
    "vectors": { ... },
    "evolution_context": "Previous reroute to Phase 5 improved failure_awareness from 0.45 to 0.63. Still below threshold. Focus on containment strategies."
  },
  "preserved_context": {
    "phases_1_through_4": "...",
    "selected_candidate": "..."
  },
  "instructions": "Structural Synthesis must address: (1) Add containment strategies for payment-order failure path. (2) Reduce coupling between notification service and core services to improve adaptability.",
  "budget_remaining": {
    "reroute_iterations": 2,
    "overdrive_available": true
  }
}
```

---

*Reroute Logic Engine v1.0 — Kernel Audit Infrastructure*
