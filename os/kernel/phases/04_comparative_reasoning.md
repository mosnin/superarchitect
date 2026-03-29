# Phase 04: Comparative Reasoning

> Score, compare, and select. Not by gut feeling -- by structured evaluation against the project-specific success model.

---

## Purpose

Comparative Reasoning takes the candidate set from Phase 3 and the success model from Phase 2, and determines which architecture direction to pursue. This is the decision phase -- the point where exploration converges toward a single structural commitment.

The phase must produce a rigorous, evidence-backed comparison that explains WHY the chosen direction won. "It felt right" is not acceptable. "It scores highest on the top 3 priority dimensions with acceptable tradeoffs on dimensions 4-6" is acceptable.

---

## Responsibilities

1. **Evaluate each candidate** against every dimension in the success model (both universal backbone and project-specific dimensions)
2. **Produce a comparative matrix** with scores, evidence strength, and uncertainty per dimension per candidate
3. **Identify tradeoff fronts** -- where candidates excel on different dimensions, creating genuine tension
4. **Rank candidates** using the tradeoff priority ordering from the success model
5. **Determine selection strategy**: select one winner OR hybridize by taking the best structural traits from multiple candidates
6. **Document the rationale**: explain the decision in terms the Success Model makes auditable
7. **Flag residual uncertainties**: what could change this decision if more information were available?

---

## Comparative Matrix

The core output of this phase. A structured comparison of every candidate against every dimension.

```yaml
comparative_matrix:
  dimensions:
    - name: "coherence"
      type: "universal_backbone"
      candidates:
        - candidate_id: "cand_event_sourced"
          score: 0.82
          evidence: 0.75
          uncertainty: 0.15
          rationale: "Clear domain boundaries but inter-service consistency requires careful orchestration"
        - candidate_id: "cand_modular_monolith"
          score: 0.88
          evidence: 0.80
          uncertainty: 0.10
          rationale: "Single deployment unit naturally coheres; module boundaries enforced by code structure"
        - candidate_id: "cand_edge_first"
          score: 0.72
          evidence: 0.60
          uncertainty: 0.25
          rationale: "Edge distribution creates coherence challenges; central coordination must be carefully designed"
    - name: "conflict_resolution"
      type: "project_specific"
      candidates:
        - candidate_id: "cand_event_sourced"
          score: 0.75
          evidence: 0.70
          uncertainty: 0.20
          rationale: "Event sourcing provides audit trail but conflict resolution is application-level concern"
        - candidate_id: "cand_modular_monolith"
          score: 0.90
          evidence: 0.85
          uncertainty: 0.10
          rationale: "CRDT core provides mathematically guaranteed conflict resolution"
        - candidate_id: "cand_edge_first"
          score: 0.80
          evidence: 0.55
          uncertainty: 0.30
          rationale: "Edge CRDTs possible but less proven at scale; merge semantics at edge add complexity"
```

---

## Scoring Rules

1. **Scores are relative to the project success model**, not to an absolute standard. A score of 0.85 means "this candidate satisfies 85% of what world-class means for THIS project on THIS dimension."

2. **Evidence must support the score**. A high score with low evidence is flagged as uncertain. The Comparative Reasoner must explain what evidence backs each score.

3. **Uncertainty must be honest**. If the Reasoner is unsure whether a candidate can deliver on a dimension, uncertainty must reflect that. Overconfident scoring is an anti-slop violation.

4. **Dimension weights come from the success model**. The tradeoff priority ordering determines which dimensions matter most in the final ranking.

---

## Ranking Method

1. Apply failure conditions first: any candidate violating a failure condition is eliminated
2. Compute weighted score per candidate using dimension weights from the success model
3. Compute confidence-adjusted score: `adjusted = score * evidence * (1 - uncertainty)`
4. Rank by confidence-adjusted weighted score
5. Check for narrow gaps: if top two candidates differ by less than 5% on adjusted score, flag for potential escalation or hybridization

---

## Hybridization

Sometimes no single candidate is clearly superior. Different candidates may excel on different high-priority dimensions. In this case, the Comparative Reasoner may recommend hybridization.

### When to hybridize

- Top two candidates score within 5% of each other on adjusted score
- Each candidate is clearly superior on different high-priority dimensions
- The structural elements that make each candidate strong are compatible (can coexist in one architecture)
- Hybridization does not introduce contradictions or incoherence

### When NOT to hybridize

- One candidate is clearly superior across most dimensions
- The structural elements from different candidates are fundamentally incompatible
- Hybridization would increase complexity without proportional quality gain
- The resulting hybrid would be harder to implement than either pure candidate

### Hybridization output

```yaml
selection:
  strategy: "hybrid"
  source_candidates:
    - candidate_id: "cand_modular_monolith"
      elements_taken:
        - "CRDT core for conflict resolution"
        - "Module boundary pattern"
    - candidate_id: "cand_event_sourced"
      elements_taken:
        - "Event sourcing for audit trail"
        - "Independent scaling for presence service"
  rationale: "CRDT monolith provides best conflict resolution and simplicity, but presence service benefits from independent scaling. Event sourcing provides audit trail without requiring full microservices decomposition."
  hybrid_risks:
    - "Complexity of maintaining both CRDT state and event store"
    - "Module boundary discipline must prevent monolith from absorbing presence service"
```

---

## OS Team Dispatch

### Architecture Team (Team 2)
Reviews candidates from a structural and domain perspective. Identifies practical implementation challenges that pure structural analysis might miss. Validates that candidate subsystem decompositions are domain-appropriate.

### Security Team (Team 7)
Reviews each candidate's attack surface, authentication architecture, and data protection posture. Security concerns can shift rankings -- a candidate with excellent performance but poor security isolation may be ranked lower for high-consequence projects.

### Data Team (Team 5)
Reviews data model implications of each candidate. Evaluates consistency guarantees, data flow patterns, storage requirements, and query patterns. Data architecture differences between candidates can be decisive.

---

## Conditional Escalation

Escalate to human operator ONLY when ALL of the following hold:

1. **Score gap is narrow**: Top candidates differ by less than 5% on adjusted score
2. **Uncertainty is high**: Average uncertainty across top candidates exceeds 0.25
3. **Consequence is high**: Project consequence level is "high" or "critical"
4. **Decision depends on preference**: The tradeoff is strategic (cost vs. speed, vendor choice, organizational alignment) rather than technical

When escalating, present:
- The comparative matrix (simplified for human consumption)
- The specific tradeoff that requires human judgment
- The kernel's recommendation with explicit uncertainty
- What information would resolve the uncertainty without escalation

---

## Fail Conditions

The phase FAILS if:

1. **Scoring is unsupported**: Scores lack evidence or rationale
2. **Ranking ignores tradeoff priority**: The winner does not reflect the success model's priority ordering
3. **Hybridization is incoherent**: The hybrid combines structurally incompatible elements
4. **Rationale is missing**: The decision cannot be explained in terms of the success model
5. **All candidates score below failure thresholds**: No viable architecture exists in the search space (reroute to Phase 3 for broader search)

---

## Example: Collaboration Platform Selection

**Ranking result**:
1. Modular Monolith with CRDT (adjusted score: 0.79)
2. Event-Sourced Microservices (adjusted score: 0.71)
3. Serverless Edge-First (adjusted score: 0.58)

**Selection**: Hybrid of Candidates 1 and 2

**Rationale**: The CRDT monolith scores highest on conflict resolution (the top project-specific dimension) and coherence. However, the event-sourced approach provides superior audit trail capabilities and allows independent scaling of the presence service, which is important for the 10M user requirement. The hybrid takes the CRDT core and modular structure from Candidate 1, adds event sourcing for audit trail and change history, and extracts presence as an independently scalable service from Candidate 2.

---

## Kernel Agent

**Primary**: Comparative Reasoner (`os/kernel/agents/comparative_reasoner.md`)

**Supporting**: Optimization Architect reviews the selection for over-engineering. Failure Mode Architect reviews for structural weaknesses in the chosen direction.

---

*Phase 04 feeds Phase 05. The selection decision is the most consequential single moment in the pipeline. A wrong selection propagates through every subsequent phase.*
