# Phase 02: Success Model Generation

> Derive what "world class" means for THIS specific project. Not a generic quality bar -- a project-tuned definition of excellence.

---

## Purpose

Every project has a different definition of world class. A payment processing API must prioritize transaction integrity and regulatory compliance. A social media feed must prioritize latency and content relevance. A medical records system must prioritize data integrity and access control. Applying the same quality bar to all of these produces mediocre architecture.

The Success Model Generator takes the compiled intent from Phase 1 and produces a project-specific quality model that Phase 4 (Comparative Reasoning) uses to score candidates and Phase 6 (Audit) uses to measure the final architecture.

---

## Universal Backbone Dimensions

These 8 dimensions are ALWAYS present in every success model. They represent structural quality that applies to any system in any domain.

| Dimension | Definition |
|-----------|-----------|
| **Coherence** | All parts of the system serve the stated objective. No orphaned components, no purpose drift. |
| **Completeness** | Every system primitive is addressed. No missing subsystems, interfaces, or flows. |
| **Internal Consistency** | No contradictions between components. Data flows are compatible. Interfaces agree on contracts. |
| **Adaptability** | The system can evolve without requiring fundamental redesign. Change is localized. |
| **Efficiency** | Resources (compute, storage, network, human attention) are used proportionally to value delivered. |
| **Failure Awareness** | The system knows how it can break and has explicit containment for each failure mode. |
| **Legibility** | A competent engineer can understand the architecture by reading it. No hidden logic. |
| **Implementability** | The architecture can be built with available resources, skills, and timeline. |

---

## Project-Specific Dimensions

Derived from the intent object. These capture what makes THIS project's quality bar unique.

### Derivation process

1. Read intent objective and desired outcomes
2. Identify domain-specific quality concerns implied by the objective
3. Read constraint list -- constraints often imply quality dimensions (e.g., regulatory constraint implies compliance dimension)
4. Read consequence rating -- high-consequence projects surface additional dimensions (e.g., auditability, rollback capability)
5. Consult Product team for business-specific success criteria
6. Consult Research team for industry benchmarks and competitive quality bars

### Example project-specific dimensions

For a **fintech payment API**:
- `transaction_integrity`: Every transaction must be exactly-once, never lost, never duplicated
- `pci_compliance`: Architecture must satisfy PCI-DSS requirements at the structural level
- `p99_latency`: 99th percentile transaction latency must be under defined threshold
- `audit_trail`: Every state change must be traceable to origin
- `regulatory_adaptability`: Architecture must accommodate new regulatory requirements without redesign

For a **real-time collaboration platform**:
- `conflict_resolution`: Concurrent edits must converge to consistent state
- `presence_latency`: User presence updates must propagate in sub-second time
- `data_durability`: No user content loss under any failure scenario
- `horizontal_scalability`: System must scale linearly with user count
- `extensibility`: New collaboration modes must be addable without core changes

---

## Tradeoff Priority Ordering

Not all dimensions can be maximized simultaneously. The success model must define a strict ordering that tells Phase 4 (Comparative Reasoning) how to break ties.

```yaml
tradeoff_priority:
  - transaction_integrity    # non-negotiable -- never sacrificed
  - pci_compliance           # regulatory requirement -- never sacrificed
  - p99_latency              # primary user experience driver
  - adaptability             # long-term viability
  - efficiency               # operational cost management
  - legibility               # maintainability
```

Rules for ordering:
1. Regulatory and safety dimensions are always top priority
2. Core user experience dimensions follow
3. Long-term viability dimensions follow
4. Operational efficiency dimensions follow
5. Aesthetic and convenience dimensions are lowest priority

---

## Failure Conditions

What makes this project FAIL? Not "what makes it imperfect" -- what makes it genuinely broken?

```yaml
failure_conditions:
  - dimension: "transaction_integrity"
    threshold: 0.95
    failure_description: "Any architecture that cannot guarantee exactly-once transaction processing is disqualified"
  - dimension: "pci_compliance"
    threshold: 1.0
    failure_description: "Non-compliant architecture cannot legally process payments"
  - dimension: "coherence"
    threshold: 0.70
    failure_description: "Incoherent architecture cannot be implemented correctly"
```

Failure conditions are hard gates. A candidate that violates any failure condition is eliminated regardless of other scores.

---

## Success Thresholds

Minimum acceptable scores per dimension. Unlike failure conditions (which eliminate), thresholds trigger rerouting in Phase 6.

```yaml
pass_logic:
  minimum_dimension_scores:
    coherence: 0.75
    completeness: 0.70
    internal_consistency: 0.80
    adaptability: 0.65
    efficiency: 0.60
    failure_awareness: 0.70
    legibility: 0.70
    implementability: 0.75
    # project-specific dimensions inherit from consequence level
  evidence_thresholds:
    minimum_evidence_per_dimension: 0.50
    minimum_average_evidence: 0.65
  uncertainty_tolerances:
    maximum_uncertainty_per_dimension: 0.40
    maximum_average_uncertainty: 0.30
```

Threshold calibration:
- **Critical consequence** projects: thresholds raised 10-15%
- **Low consequence** projects: thresholds may be lowered 5-10%
- Evidence and uncertainty thresholds remain constant regardless of consequence

---

## Evidence and Uncertainty Requirements

Every score in the success model must be backed by evidence and carry an uncertainty rating. This prevents the kernel from producing overconfident assessments.

- **Evidence** (0.0-1.0): How much structural analysis supports the score? A score of 0.9 with evidence 0.3 means "we think it is good but we have not actually checked."
- **Uncertainty** (0.0-1.0): How much could this score change with more analysis? High uncertainty flags dimensions that need deeper audit.

The success model defines minimum evidence and maximum uncertainty thresholds. Dimensions that fail evidence thresholds are automatically flagged for deeper analysis in Phase 6.

---

## Domain Context

The Success Model Architect may consult domain practitioners to ground project-specific dimensions in reality:
- **Domain experts** provide field-specific success criteria that structural analysis alone cannot derive — clinical outcome benchmarks, regulatory thresholds, logistics performance norms, or customer experience standards.
- **Benchmarks and standards** from the target domain set the baseline for what "world class" means. A structurally correct but domain-ignorant success model produces thresholds that are either trivially easy or impossible.

Domain context is optional input — the kernel can produce a valid success model without it, but the resulting thresholds will carry higher uncertainty.

---

## Fail Conditions

The phase FAILS if:

1. **Dimensions are generic to the point of uselessness**: If the success model for a payment API is identical to the success model for a blog, it has failed to capture project specificity.

2. **Tradeoffs are unresolved**: If the model cannot order dimensions by priority, Phase 4 cannot make selection decisions.

3. **Thresholds are non-actionable**: If thresholds are either so low that everything passes or so high that nothing can pass, they provide no signal.

4. **Failure conditions are missing**: Every project has ways to catastrophically fail. If the model does not capture them, it is incomplete.

---

## Output Schema

```yaml
success_model:
  universal_backbone:
    coherence:
      threshold: 0.75
      weight: 1.0
    completeness:
      threshold: 0.70
      weight: 0.9
    internal_consistency:
      threshold: 0.80
      weight: 1.0
    adaptability:
      threshold: 0.65
      weight: 0.8
    efficiency:
      threshold: 0.60
      weight: 0.7
    failure_awareness:
      threshold: 0.70
      weight: 0.9
    legibility:
      threshold: 0.70
      weight: 0.8
    implementability:
      threshold: 0.75
      weight: 0.9
  project_specific_dimensions:
    "<dimension_name>":
      definition: "<what this dimension measures>"
      threshold: 0.0-1.0
      weight: 0.0-1.0
      rationale: "<why this dimension matters for this project>"
  tradeoff_priority:
    - "<highest priority dimension>"
    - "<next priority>"
  failure_conditions:
    - dimension: "<dimension>"
      threshold: 0.0-1.0
      failure_description: "<what happens if this fails>"
  pass_logic:
    minimum_dimension_scores: {}
    evidence_thresholds:
      minimum_evidence_per_dimension: 0.50
      minimum_average_evidence: 0.65
    uncertainty_tolerances:
      maximum_uncertainty_per_dimension: 0.40
      maximum_average_uncertainty: 0.30
```

---

## Kernel Agent

**Primary**: Success Model Architect (`os/kernel/agents/success_model_architect.md`)

---

*Phase 02 feeds Phases 04 and 06. Without a rigorous success model, candidate comparison is subjective and audit is unmeasurable.*
