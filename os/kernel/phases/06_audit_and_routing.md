# Phase 06: Audit and Routing

> Measure with vectors. Route to the weakest layer. Fix surgically, not wholesale.

---

## Purpose

Audit and Routing is the kernel's quality enforcement engine. It takes the synthesized architecture from Phase 5, measures it against the success model from Phase 2, and produces machine-readable audit vectors. If any dimension falls below threshold, the kernel routes the package back to the shallowest stage capable of fixing the defect.

This phase is not a rubber stamp. It is the mechanism that makes kernel output genuinely world-class rather than merely plausible. Without it, the kernel is just a generator. With it, the kernel is a self-correcting optimization system.

---

## Audit Vector

For each dimension in the success model (both universal backbone and project-specific), the audit emits a structured vector:

```yaml
audit_vector:
  dimensions:
    "<dimension_name>":
      score: 0.0-1.0          # quality rating for this dimension
      evidence: 0.0-1.0       # how much structural analysis supports the score
      uncertainty: 0.0-1.0    # how much the score could change with more analysis
      delta: -1.0 to 1.0      # change from previous audit pass (0.0 on first pass)
      status: "pass|warn|fail|reroute"
      rationale: "<why this score>"
      evidence_sources:
        - "<what was examined to produce this score>"
      reroute_target: "<phase to reroute to if status is reroute>"
```

### Score semantics
- 0.0-0.39: Critical failure. Architecture is fundamentally broken on this dimension.
- 0.40-0.59: Significant weakness. Architecture has structural gaps requiring correction.
- 0.60-0.74: Below threshold. Architecture needs targeted improvement.
- 0.75-0.89: Acceptable. Architecture meets baseline quality.
- 0.90-1.0: Excellent. Architecture excels on this dimension.

### Evidence semantics
- 0.0-0.29: Claim with no supporting analysis. Score is unreliable.
- 0.30-0.49: Weak evidence. Score is directionally correct but not verified.
- 0.50-0.74: Moderate evidence. Score is supported by structural analysis.
- 0.75-1.0: Strong evidence. Score is supported by detailed analysis with specific references.

### Uncertainty semantics
- 0.0-0.14: High confidence. Unlikely to change with more analysis.
- 0.15-0.29: Moderate confidence. Could shift by 0.1 with deeper analysis.
- 0.30-0.49: Uncertain. Could shift significantly with more information.
- 0.50-1.0: Highly uncertain. Score is essentially a guess.

---

## Universal Backbone Audit

These 8 dimensions are always audited, regardless of project type.

### 1. Coherence
Do all subsystems serve the stated objective? Are there orphaned components? Does the architecture tell a single coherent story?

### 2. Completeness
Are all 11 system primitives represented? Are there missing subsystems, unspecified interfaces, or incomplete flows?

### 3. Internal Consistency
Do interface contracts agree between connected subsystems? Are data formats compatible? Do timing assumptions align? Are there contradictions?

### 4. Adaptability
Can the system evolve without fundamental redesign? Are extension points defined? Can new capability be added by adding components rather than modifying existing ones?

### 5. Efficiency
Are resources used proportionally to value? Is there unnecessary duplication? Are expensive operations justified by their contribution to system goals?

### 6. Failure Awareness
Does the system know how it can break? Is every failure mode identified with containment? Are blast radii bounded? Does the system degrade gracefully?

### 7. Legibility
Can a competent engineer understand the architecture by reading the synthesis? Are boundaries clear? Are flows traceable? Is naming consistent and meaningful?

### 8. Implementability
Can this architecture be built with available resources, skills, and timeline? Are technology choices realistic? Are team skill requirements acknowledged?

---

## Pass Logic

The audit passes when ALL of the following conditions hold:

1. **All dimensions above threshold**: Every dimension's score meets or exceeds the minimum defined in the success model
2. **No failure conditions violated**: No dimension falls below the hard failure threshold
3. **Evidence above minimum**: Average evidence across all dimensions exceeds the evidence threshold. No individual dimension has evidence below 0.30
4. **Uncertainty below maximum**: Average uncertainty across all dimensions is below the uncertainty tolerance. No individual dimension has uncertainty above 0.50
5. **Anti-slop check passes**: The architecture does not exhibit any anti-slop violations (vague, overabstract, checklist-heavy, contradictory, underaudited, overconfident, too rigid)

If ANY condition fails, the audit triggers a reroute.

---

## Reroute Logic

The kernel routes back to the SHALLOWEST stage that can fix the defect. This is critical -- rerouting too deep wastes work, but rerouting too shallow fails to fix the problem.

### Reroute Map

| Weak Area | Symptom | Reroute Target | Rationale |
|-----------|---------|----------------|-----------|
| **Poor candidate diversity** | Winner was selected from insufficiently varied candidates | Phase 3 (Architecture Search) | Need more structural theses to explore |
| **Weak ranking logic** | Selection rationale is unconvincing or contradictory | Phase 4 (Comparative Reasoning) | Re-score with better evidence |
| **Weak interfaces** | Contracts between subsystems are unclear or incompatible | Phase 5 (Structural Synthesis) | Interface design is a synthesis responsibility |
| **Missing subsystems** | Required functionality has no home in the architecture | Phase 5 (Structural Synthesis) | Subsystem decomposition needs revision |
| **Weak failure containment** | Blast radii are unbounded, no graceful degradation | Phase 5 (Structural Synthesis) | Failure architecture is a synthesis responsibility |
| **Weak evidence** | Scores are high but evidence is low | Phase 6 (re-audit with deeper analysis) | Need more thorough examination, not redesign |
| **Unresolved ambiguity** | Intent gaps are causing architectural uncertainty | Phase 1 (Intent Compilation) | Fundamental requirements are unclear |
| **Wrong success criteria** | Architecture is well-built but for the wrong goals | Phase 2 (Success Model) | Quality model does not match project needs |
| **Low adaptability** | No evolution paths, rigid structure | Phase 5 (Structural Synthesis) | Need to add extension points and migration paths |
| **Incoherence** | Subsystems work against each other | Phase 5 (Structural Synthesis) or Phase 3 if fundamental | Depends on whether the thesis itself is contradictory |

### Reroute rules

1. Always route to the shallowest stage that can fix the defect
2. Never discard work from stages that passed audit
3. Carry the audit vector forward so the rerouted stage knows exactly what to fix
4. Maximum reroute depth: 3 iterations per dimension before escalation
5. If the same dimension fails 3 times, escalate to human with full audit history

---

## Evolution Ledger

Every reroute appends an immutable entry to the evolution ledger in the canonical package. This creates a traceable history of how the architecture evolved through corrective iteration.

```yaml
evolution:
  - iteration: 1
    timestamp: "2026-03-29T10:15:00Z"
    trigger: "audit_reroute"
    failed_dimensions:
      adaptability:
        score: 0.55
        threshold: 0.65
        delta: 0.0
    reroute_target: "phase_05_structural_synthesis"
    mutation_applied: "Added extension points for new collaboration modes; defined migration path from single-region to multi-region; introduced plugin interface for document type handlers"
    rationale: "Architecture had no evolution paths defined. All change required core modification."
    result: "adaptability score improved from 0.55 to 0.78"
```

The ledger is never modified or deleted. It is append-only. This ensures the kernel can always explain why the architecture looks the way it does.

---

## Overdrive Mode

Activated when one or more dimensions fall below a critical threshold (typically 0.40). Overdrive is not a general mode -- it is a focused intervention.

### Overdrive triggers
- Any dimension scores below 0.40
- Three or more dimensions score between 0.40 and 0.60
- Evidence is below 0.30 on any dimension with score below 0.70

### Overdrive behavior
1. Identify the weakest dimensions
2. For each weak dimension, spawn a focused specialist pass:
   - Failure awareness weakness: dispatch Failure Mode Architect + Security team for deep failure analysis
   - Adaptability weakness: dispatch Synthesis Architect + Architecture team for evolution path design
   - Efficiency weakness: dispatch Optimization Architect for simplification analysis
   - Coherence weakness: dispatch Controller Architect for objective alignment review
3. Each specialist pass produces a targeted mutation
4. Mutations are applied by the Mutation Architect
5. Re-audit after mutations

Overdrive deactivates when all dimensions are above the critical threshold.

---

## Kernel Agent

**Primary**: Audit Architect — owns all confidence vector production, dimension scoring, reroute targeting, and evolution ledger entries.

The Audit Architect operates independently from the Synthesis Architect to prevent self-review bias. It receives the synthesized architecture as input and evaluates it against the success model without knowledge of the synthesis decisions that produced it.

Domain practitioners may provide domain-specific evidence for project-specific dimensions (e.g., regulatory compliance assessment, clinical safety review, load-bearing verification) — but the structural dimensions of the universal backbone are scored by the kernel alone.

---

## Fail Conditions for the Audit Phase Itself

The audit phase can fail in its OWN execution (distinct from the architecture failing audit):

1. **Scores without evidence**: Audit produces numbers without explaining what was examined
2. **Missing dimensions**: Not all success model dimensions are audited
3. **Reroute without target**: Audit identifies weakness but does not specify which stage can fix it
4. **Inconsistent vector**: Scores contradict each other (e.g., high coherence but low internal consistency)
5. **No delta tracking**: On re-audit after reroute, deltas are not computed

---

## Example: Audit Vector with Reroute

```yaml
audit_vector:
  dimensions:
    coherence:
      score: 0.85
      evidence: 0.80
      uncertainty: 0.10
      delta: 0.0
      status: "pass"
      rationale: "All subsystems serve the collaboration objective. No orphaned components."
    adaptability:
      score: 0.55
      evidence: 0.75
      uncertainty: 0.15
      delta: 0.0
      status: "reroute"
      rationale: "No extension points defined. Adding new document types requires modifying core CRDT engine. No migration path for multi-region deployment."
      reroute_target: "phase_05_structural_synthesis"
    failure_awareness:
      score: 0.78
      evidence: 0.70
      uncertainty: 0.20
      delta: 0.0
      status: "pass"
      rationale: "Failure modes identified for most subsystems. Presence service failure containment is well-defined. Event store durability strategy is sound."
    conflict_resolution:
      score: 0.92
      evidence: 0.90
      uncertainty: 0.05
      delta: 0.0
      status: "pass"
      rationale: "CRDT core provides mathematically proven convergence. Operational transform fallback for complex operations is specified."
routing_decision:
  action: "reroute"
  target: "phase_05_structural_synthesis"
  instructions: "Add extension points for document type handlers. Define migration path from single-region to multi-region. Introduce plugin interface for CRDT engine."
  preserve: ["All existing subsystem definitions", "Interface contracts", "Flow definitions", "Failure containment"]
```

---

## Kernel Agent

**Primary**: Audit Architect (`os/kernel/agents/audit_architect.md`)

**Supporting**: Failure Mode Architect provides adversarial analysis. Mutation Architect applies corrective changes during reroute.

---

*Phase 06 either advances to Phase 07 (Packaging) or routes back to an earlier phase. The kernel does not ship weak architecture. It fixes it or escalates.*
