# Universal Audit Backbone

> The 8 dimensions every system is audited against, regardless of domain.

---

## Purpose

The Universal Audit Backbone defines the structural quality dimensions that apply to ANY system — software, organizational, physical, or conceptual. These dimensions are domain-agnostic. They measure whether a system is well-formed, not whether it solves a particular problem well. Domain-specific quality is layered on top by domain packs (see `integration/domain_bridge.md`).

Every system that passes through the kernel's Phase 6 (Audit) is scored against all 8 dimensions. A system that fails any dimension is rerouted to the shallowest phase capable of fixing the deficiency (see `reroute_logic.md`).

---

## Dimension 1: Coherence

**Definition**: Do all parts of the system work together toward the stated objective? A coherent system has no orphaned components, no contradictory subsystems, and no parts that pull in different directions.

**Measurement Method**:
- Cross-reference analysis: trace every component back to the stated purpose. Components that cannot be traced are incoherent.
- Interface consistency: verify that connected components agree on data formats, protocols, and semantics.
- Flow completeness: trace every input through the system to its output. Incomplete flows indicate incoherence.
- Goal alignment: verify each subsystem's local objective contributes to the global objective.

**Scoring Guide**:
- 0.9 - 1.0: All components trace to purpose. All interfaces consistent. All flows complete.
- 0.7 - 0.9: Minor orphaned components or minor interface mismatches. Fixable without rearchitecting.
- 0.5 - 0.7: Significant misalignment. Some subsystems serve unclear purposes. Reroute required.
- Below 0.5: Fundamental incoherence. System parts contradict each other. Reroute to Phase 1 (Intent).

**Evidence Requirements**: Component-to-purpose trace map. Interface contract comparison matrix. Flow completion report.

**Common Failure Modes**: Feature creep adding unrelated components. Copy-paste integration without adaptation. Multiple teams building without shared understanding of purpose.

**Reroute Target**: Phase 1 (Intent Compilation) if purpose is unclear. Phase 5 (Structural Synthesis) if structure is incoherent but purpose is clear.

---

## Dimension 2: Completeness

**Definition**: Are all required elements present? A complete system has every primitive defined, every interface specified, every flow documented, and every edge case addressed.

**Measurement Method**:
Checklist of system primitives — score = percentage present and well-defined:
1. Purpose — clearly stated objective
2. Boundary — what is inside vs outside the system
3. Inputs — all ingress points enumerated
4. Transformations — all processing logic specified
5. Outputs — all egress points enumerated
6. Interfaces — all connection points defined with contracts
7. Resources — all dependencies identified
8. Constraints — all limitations documented
9. Feedback — monitoring and adjustment mechanisms
10. Failure Modes — what can go wrong and how system responds
11. Evolution Paths — how the system changes over time

**Scoring Guide**:
- 0.9 - 1.0: All 11 primitives present with full definitions.
- 0.7 - 0.9: 8-10 primitives present. Missing ones are non-critical or inferable.
- 0.5 - 0.7: 6-8 primitives present. Gaps create ambiguity. Reroute required.
- Below 0.5: Fewer than 6 primitives defined. System is a sketch, not a design.

**Evidence Requirements**: Primitive checklist with status for each. Gap analysis for missing primitives.

**Common Failure Modes**: Skipping failure modes ("happy path only"). Omitting evolution paths. Defining inputs but not outputs. Ignoring constraints.

**Reroute Target**: Phase 5 (Structural Synthesis) for missing structural primitives. Phase 2 (Success Model) if success criteria themselves are incomplete.

---

## Dimension 3: Internal Consistency

**Definition**: Do components agree with each other? A consistent system has no contradictory assumptions, no interface mismatches, and no circular dependencies without explicit termination conditions.

**Measurement Method**:
- Assumption audit: extract implicit assumptions from each component. Flag contradictions.
- Interface contract matching: for every producer-consumer pair, verify the producer's output schema matches the consumer's input expectation.
- Data flow analysis: verify acyclicity or confirm that cycles have explicit termination/convergence conditions.
- Naming consistency: verify the same concept uses the same name everywhere.

**Scoring Guide**:
- 0.9 - 1.0: Zero contradictions. All contracts match. All naming consistent.
- 0.7 - 0.9: Minor inconsistencies that are resolvable without structural changes.
- 0.5 - 0.7: Multiple contradictory assumptions or interface mismatches. Reroute required.
- Below 0.5: Fundamental contradictions. Components built on incompatible assumptions.

**Evidence Requirements**: Assumption contradiction report. Interface contract diff. Naming consistency audit.

**Common Failure Modes**: Different teams using different terminology for the same concept. Stale interface contracts after changes. Implicit assumptions never documented.

**Reroute Target**: Phase 5 (Structural Synthesis) to reconcile structural inconsistencies. Phase 3 (Architecture Search) if inconsistencies stem from fundamentally incompatible architectural choices.

---

## Dimension 4: Adaptability

**Definition**: Can the system evolve without being rebuilt? An adaptable system has loose coupling, explicit extension points, versioned interfaces, and documented evolution paths.

**Measurement Method**:
- Loose coupling score: measure the ratio of inter-component dependencies to total components. Lower is better.
- Change impact analysis: for a hypothetical change to each component, count how many other components are affected.
- Versioning strategy: verify interfaces have versioning and backward compatibility strategy.
- Evolution paths: verify the system documents how it will handle growth, technology changes, and requirement shifts.

**Scoring Guide**:
- 0.9 - 1.0: Low coupling. Small change blast radius. Versioned interfaces. Clear evolution paths.
- 0.7 - 0.9: Moderate coupling. Most changes are contained. Evolution paths partially documented.
- 0.5 - 0.7: High coupling. Changes cascade. No versioning. Evolution not considered.
- Below 0.5: Monolithic. Any change risks the whole system. No path to evolution.

**Evidence Requirements**: Coupling matrix. Change impact analysis for 3 representative changes. Evolution path documentation.

**Common Failure Modes**: Tight coupling disguised as "simplicity." No versioning because "we'll deal with it later." Evolution paths that assume the current architecture is permanent.

**Reroute Target**: Phase 5 (Structural Synthesis) to improve boundaries and coupling. Phase 3 (Architecture Search) if the chosen architecture is fundamentally rigid.

---

## Dimension 5: Efficiency

**Definition**: Is the system unnecessarily complex? An efficient system achieves its purpose with the minimum necessary structure. Every component earns its existence.

**Measurement Method**:
- Component count vs. minimum needed: could any component be removed without losing required functionality?
- Dependency depth: how deep is the longest dependency chain? Deeper = more fragile.
- Abstraction audit: is every abstraction justified by actual variation, or are there speculative abstractions?
- Redundancy check: are there duplicate components doing the same thing differently?

**Scoring Guide**:
- 0.9 - 1.0: Every component is necessary. No speculative abstractions. Minimal dependency depth.
- 0.7 - 0.9: Minor redundancy or 1-2 speculative abstractions. Easily pruned.
- 0.5 - 0.7: Significant over-engineering. Multiple unnecessary layers. Reroute required.
- Below 0.5: Architecture is dominated by unnecessary complexity. Fundamental simplification needed.

**Evidence Requirements**: Component necessity justification. Dependency depth report. Abstraction justification audit.

**Common Failure Modes**: Premature abstraction. "Enterprise patterns" applied to simple problems. Multiple caching layers without measured need. Microservices where a monolith suffices.

**Reroute Target**: Phase 5 (Structural Synthesis) to simplify. Phase 3 (Architecture Search) if the architectural style itself is the source of over-complexity.

---

## Dimension 6: Failure Awareness

**Definition**: Does the system know how it can break? A failure-aware system identifies its failure modes, contains failures to prevent cascading, and defines recovery paths.

**Measurement Method**:
- Failure mode enumeration: count identified failure modes vs. expected failure modes (based on component count and integration points).
- Containment strategy coverage: what percentage of failure modes have explicit containment strategies?
- Recovery path coverage: what percentage of failure modes have defined recovery procedures?
- Cascading failure analysis: are there failure chains that can take down the entire system?

**Scoring Guide**:
- 0.9 - 1.0: All plausible failure modes identified. Each has containment and recovery. No uncontained cascading paths.
- 0.7 - 0.9: Most failure modes identified. Minor gaps in recovery paths. No critical cascading risks.
- 0.5 - 0.7: Many failure modes unidentified. Limited containment. Some cascading risks.
- Below 0.5: Failure modes not systematically considered. No containment. System is fragile.

**Evidence Requirements**: Failure mode registry. Containment strategy map. Recovery path documentation. Cascading failure analysis.

**Common Failure Modes**: Only considering infrastructure failures, not logic failures. No timeout strategy. No circuit breakers. Assuming dependencies are always available.

**Reroute Target**: Phase 5 (Structural Synthesis) to add failure handling. Phase 3 (Architecture Search) if the architecture lacks structural support for failure containment (e.g., no bulkheads).

---

## Dimension 7: Legibility

**Definition**: Can a new person understand this system? A legible system has clear naming, documented intent, traceable flows, and minimal hidden complexity.

**Measurement Method**:
- Naming clarity: are component names self-explanatory? Do they describe what, not how?
- Documentation quality: is architectural intent documented? Are non-obvious decisions explained?
- Flow traceability: can a reader trace a request from entry to exit without guesswork?
- Hidden complexity: are there implicit behaviors, side effects, or magic values that a reader would not expect?

**Scoring Guide**:
- 0.9 - 1.0: A competent engineer can understand the system in under 1 hour of reading. All decisions documented.
- 0.7 - 0.9: Mostly clear. A few areas require asking questions or reading source code.
- 0.5 - 0.7: Significant areas are opaque. Tribal knowledge required. Reroute required.
- Below 0.5: System is incomprehensible without the original author. Documentation absent or misleading.

**Evidence Requirements**: Naming audit. Documentation coverage report. Flow trace walkthrough. Hidden complexity inventory.

**Common Failure Modes**: Clever naming that only makes sense to the author. Documentation that describes the code but not the decisions. Diagrams that are out of date.

**Reroute Target**: Phase 7 (Packaging) for documentation gaps. Phase 5 (Structural Synthesis) if the structure itself is illegible (poor boundaries, tangled flows).

---

## Dimension 8: Implementability

**Definition**: Can this system actually be built with available resources, technology, and constraints? An implementable system is feasible, not just elegant.

**Measurement Method**:
- Technology feasibility: are all required technologies mature, available, and well-understood?
- Skill availability: does the expected team have (or can acquire) the skills needed?
- Effort estimation: is the estimated effort realistic given the timeline and resources?
- Dependency availability: are all external dependencies available, maintained, and appropriately licensed?
- Regulatory feasibility: does the design comply with applicable regulations?

**Scoring Guide**:
- 0.9 - 1.0: All technologies proven. Team has skills. Effort is realistic. Dependencies available. Regulations met.
- 0.7 - 0.9: Minor skill gaps or 1-2 unproven technologies, but mitigable.
- 0.5 - 0.7: Significant feasibility risks. Unproven technologies in critical paths. Timeline at risk.
- Below 0.5: Design is aspirational, not practical. Major technology or skill gaps. Reroute required.

**Evidence Requirements**: Technology maturity assessment. Skill gap analysis. Effort estimate with confidence range. Dependency audit. Regulatory compliance checklist.

**Common Failure Modes**: Choosing cutting-edge technology without evaluating maturity. Underestimating integration effort. Ignoring licensing restrictions. Designing for a team that does not exist.

**Reroute Target**: Phase 5 (Structural Synthesis) to simplify the design. Phase 3 (Architecture Search) to find a more feasible architectural approach. Phase 1 (Intent Compilation) if the stated objective is fundamentally infeasible.

---

## Audit Execution Protocol

1. Score all 8 dimensions independently. Do not let a high score in one dimension compensate for a low score in another.
2. For each dimension, produce a confidence vector (see `confidence_vector.md`).
3. Flag any dimension below threshold for rerouting (see `threshold_logic.md`).
4. Record all scores in the evolution ledger (see `evolution_ledger.md`).
5. Compute the reroute target for any failing dimension (see `reroute_logic.md`).
6. Return the complete audit result to the Controller Loop.

## Cross-Dimension Interactions

Some dimensions have natural tensions:
- **Efficiency vs. Adaptability**: Maximum efficiency may reduce flexibility. Balance required.
- **Efficiency vs. Failure Awareness**: Failure handling adds components. Accept the complexity it brings.
- **Completeness vs. Implementability**: A fully complete design may be too ambitious. Scope must be feasible.

When tensions are detected, document the trade-off explicitly in the evolution ledger. Do not silently favor one dimension over another.

---

*Universal Audit Backbone v1.0 — Kernel Audit Infrastructure*
