# Runtime Modes

> Different projects demand different levels of rigor. Modes tune the kernel's behavior.

---

## Overview

Runtime modes control how the kernel balances exploration depth, audit strictness, and resource consumption. The default mode is appropriate for most projects. Other modes exist for high-stakes, novel, or resource-constrained scenarios.

Modes affect three things:
1. **Candidate count**: How many architectural candidates are generated in Phase 3.
2. **Audit thresholds**: How strict the pass/fail criteria are (see `threshold_logic.md`).
3. **Reroute behavior**: How aggressively the system corrects weaknesses.

---

## Mode: Standard

**When to use**: Most projects. The system type is understood. The domain is familiar. The consequences of architectural mistakes are significant but not catastrophic.

**Configuration**:
```json
{
  "mode": "standard",
  "candidates": 3,
  "score_threshold": 0.75,
  "evidence_threshold": 0.60,
  "uncertainty_max": 0.30,
  "max_reroute_iterations": 5,
  "overdrive_trigger": 0.50,
  "escalation_after_stagnant_iterations": 3
}
```

**Behavior**:
- 3 candidate architectures generated and compared.
- Standard audit thresholds (0.75 score, 0.6 evidence, 0.3 uncertainty).
- Normal reroute budget (5 iterations).
- Balanced resource consumption.

---

## Mode: Search Heavy

**When to use**: High-consequence projects where the wrong architecture is expensive to change. Novel domains where the right approach is genuinely unknown. Greenfield projects with no precedent.

**Configuration**:
```json
{
  "mode": "search_heavy",
  "candidates": 5,
  "candidate_diversity_min": 0.6,
  "score_threshold": 0.75,
  "evidence_threshold": 0.65,
  "uncertainty_max": 0.30,
  "max_reroute_iterations": 7,
  "overdrive_trigger": 0.50,
  "escalation_after_stagnant_iterations": 3,
  "deep_comparison": true
}
```

**Behavior**:
- 5+ candidate architectures generated. Minimum diversity threshold enforced — candidates must be structurally different, not minor variations.
- Deep comparison in Phase 4: each candidate is evaluated against all 8 audit dimensions (not just the top-level comparison criteria).
- Slightly higher evidence threshold (0.65) to ensure comparisons are well-grounded.
- Larger reroute budget (7 iterations) to allow for more exploration.
- Slower convergence but higher confidence in the selected architecture.
- Consumes approximately 2x the resources of Standard mode.

---

## Mode: Conservative

**When to use**: Safety-critical systems (medical devices, financial infrastructure, aviation). Compliance-heavy environments (HIPAA, SOC2, PCI-DSS). Systems where failure has legal or safety consequences.

**Configuration**:
```json
{
  "mode": "conservative",
  "candidates": 3,
  "score_threshold": 0.85,
  "evidence_threshold": 0.75,
  "uncertainty_max": 0.20,
  "max_reroute_iterations": 7,
  "overdrive_trigger": 0.60,
  "escalation_after_stagnant_iterations": 2,
  "mandatory_dimensions": ["failure_awareness", "internal_consistency", "implementability"],
  "escalation_on_warn": true
}
```

**Behavior**:
- Higher audit thresholds across the board (0.85 score, 0.75 evidence, 0.20 uncertainty).
- Mandatory dimensions: failure_awareness, internal_consistency, and implementability MUST score above threshold. No exceptions, no trade-offs.
- Escalation on "warn" status — borderline results are not acceptable.
- Overdrive triggers at a higher score (0.60 instead of 0.50) — problems are caught earlier.
- Escalation after only 2 stagnant iterations (vs. 3 in Standard).
- More frequent human touchpoints for validation.

---

## Mode: Overdrive

**When to use**: NOT selected by the operator. Overdrive is activated automatically by the Controller Loop when a specific dimension falls below the overdrive trigger threshold after rerouting.

**Activation condition**: Any dimension score < `overdrive_trigger` (default 0.50) after 2+ reroute attempts targeting that dimension.

**Configuration** (applied as overlay on current mode):
```json
{
  "mode": "overdrive",
  "target_dimension": "<the failing dimension>",
  "specialist_passes": 3,
  "focused_analysis": true,
  "mutation_candidates": 3,
  "resource_multiplier": 2.0
}
```

**Behavior**:
- Spawns a focused specialist pass ONLY on the failing dimension.
- The specialist pass:
  1. Performs deep-dive root cause analysis on why the dimension is failing.
  2. Generates 3 targeted mutation candidates (specific structural changes, not full re-architecture).
  3. Evaluates each mutation's predicted impact on the failing dimension AND its predicted impact on other dimensions (to avoid cascade regression).
  4. Selects the mutation with the best predicted net improvement.
  5. Applies the mutation and re-audits.
- Consumes 2x normal phase resources for the focused dimension.
- Overdrive is temporary — once the dimension passes or escalation occurs, the mode reverts to the previous mode.
- Overdrive cannot be triggered more than twice per build (safety valve).

---

## Mode Selection Logic

The Controller selects the initial mode based on:

```
FUNCTION select_mode(build_request):

  IF build_request.mode IS explicitly specified:
    RETURN build_request.mode

  consequence = assess_consequence_level(build_request)
  # consequence: "low", "moderate", "high", "critical"

  novelty = assess_domain_novelty(build_request)
  # novelty: "familiar", "somewhat_novel", "novel", "unprecedented"

  IF consequence == "critical" OR compliance_requirements_present:
    RETURN "conservative"

  IF novelty IN ["novel", "unprecedented"]:
    RETURN "search_heavy"

  IF consequence == "high" AND novelty == "somewhat_novel":
    RETURN "search_heavy"

  RETURN "standard"
```

### Consequence Assessment Factors
- User count (more users = higher consequence)
- Financial impact of failure
- Safety implications
- Regulatory requirements
- Data sensitivity
- Reversibility of deployment

### Novelty Assessment Factors
- Existence of reference architectures
- Team familiarity with the domain
- Availability of proven technology stack
- Similarity to previously built systems

---

## Mode Switching

Modes can change mid-pipeline. The Controller switches modes when:

| Trigger | From | To | Rationale |
|---|---|---|---|
| Audit reveals critical weakness after reroute | Standard | Overdrive (for that dimension) | Targeted recovery needed |
| Phase 3 produces homogeneous candidates | Standard | Search Heavy | Need more exploration |
| Phase 4 reveals all candidates have low feasibility | Any | Conservative | Must be more careful about what we select |
| Human operator requests mode change | Any | Any | Explicit override |
| Budget running low | Search Heavy | Standard | Conserve remaining resources |

**Mode switch rules**:
- Mode switches are logged in the Controller's transition log.
- Thresholds change immediately upon switch. Previously passing dimensions are NOT re-audited against new thresholds unless the Controller explicitly requests it.
- Overdrive mode is always additive (overlays on current mode) and always temporary.

---

## Mode Comparison Summary

| Property | Standard | Search Heavy | Conservative | Overdrive |
|---|---|---|---|---|
| Candidates | 3 | 5+ | 3 | N/A (mutation) |
| Score threshold | 0.75 | 0.75 | 0.85 | Inherited |
| Evidence threshold | 0.60 | 0.65 | 0.75 | Inherited |
| Uncertainty max | 0.30 | 0.30 | 0.20 | Inherited |
| Max reroutes | 5 | 7 | 7 | Inherited |
| Resource usage | 1x | 2x | 1.5x | +2x per dimension |
| Escalation sensitivity | Normal | Normal | High | N/A |
| Activation | Manual/auto | Manual/auto | Manual/auto | Automatic only |

---

*Runtime Modes v1.0 — Kernel Runtime Engine*
