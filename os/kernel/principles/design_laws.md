# Design Laws

> The 10 laws that govern how the kernel operates. Non-negotiable. Every agent, every phase, every decision.

---

## Law 1: Search Before Committing

Never commit to the first architecture that comes to mind. Generate multiple genuinely different structural theses. Compare them against the project-specific success model. Select based on evidence, not intuition.

**Why**: The first idea anchors thinking. Without structured search, architects optimize locally around their initial intuition instead of exploring the global design space. Phase 3 exists to enforce this law.

---

## Law 2: Separate Structural Cognition from Domain Cognition

The kernel reasons about system structure using universal primitives. Domain expertise comes from specialist teams dispatched within kernel phases. The kernel never hardcodes domain-specific knowledge into its reasoning engine.

**Why**: Mixing structural and domain reasoning produces ad-hoc heuristics that work for one domain but fail for others. Separation makes the kernel genuinely universal.

---

## Law 3: Audit with Metrics, Not Prose Alone

Every quality assessment must produce machine-readable vectors: score, evidence, uncertainty, delta, status. Prose explanations accompany vectors but do not replace them. "The architecture is good" is not an audit result. `{score: 0.82, evidence: 0.75, uncertainty: 0.15}` is.

**Why**: Prose-only evaluation is subjective, non-comparable, and impossible to track across iterations. Vectors enable measurement, comparison, and improvement tracking.

---

## Law 4: Reroute Selectively

When audit identifies weakness, route back to the shallowest stage that can fix the defect. Never discard work from passing stages. Never reroute deeper than necessary. Carry the audit vector forward so the rerouted stage knows exactly what to fix.

**Why**: Wholesale rerouting wastes work, destroys passing structure, and resets iteration progress. Selective rerouting is surgical: it fixes the problem and preserves everything else.

---

## Law 5: Keep One Canonical Package

Every project has exactly one system package YAML. All phases read from and write to this package. No phase invents hidden state outside the package. If intermediate artifacts exist, the package holds references to them.

**Why**: Without a single source of truth, state fragments across multiple files, creating inconsistency, stale data, and untraceable decisions. One package means one truth.

---

## Law 6: Prefer Legible Complexity Over Hidden Complexity

When complexity is necessary, make it visible and understandable. Explicit interface contracts, documented dependencies, named failure modes, and traced flows are better than implicit magic, hidden coupling, and undocumented conventions.

**Why**: Hidden complexity is indistinguishable from absence of complexity until it causes a failure. Legible complexity can be reasoned about, tested, and improved.

---

## Law 7: World-Class Must Be Improvable

A world-class system is not a finished system. It is a system that can be improved without being redesigned. Extension points, migration paths, deprecation strategies, and evolution ledgers are not luxuries -- they are requirements.

**Why**: Every system changes. Systems without designed evolution paths become legacy. Systems with evolution paths remain competitive.

---

## Law 8: Every Major Claim Carries Evidence and Uncertainty

No score, recommendation, or architectural assertion exists without evidence strength and uncertainty rating. "This architecture is highly adaptable" must be accompanied by what evidence supports that claim and how much the assessment could change.

**Why**: Overconfident assertions without evidence are the primary source of architectural mistakes. Explicit uncertainty drives appropriate caution and deeper analysis where needed.

---

## Law 9: Escalate Only When Justified

The kernel defaults to full autonomy. Human escalation occurs only when uncertainty is high, consequence is high, AND the decision depends on human preference rather than technical fitness. Routine technical decisions are never escalated.

**Why**: Unnecessary escalation slows the pipeline, creates dependency on human availability, and wastes human attention on decisions the kernel is qualified to make.

---

## Law 10: Preserve Evolution History

The kernel records how the architecture evolved: every reroute, every mutation, every mode switch, every escalation. The evolution ledger is append-only and immutable. The kernel does not just know what it concluded -- it knows how it got there.

**Why**: Evolution history enables learning, debugging, and trust. An architecture without history is an architecture without explanation. When someone asks "why does the system look this way?", the evolution ledger provides the answer.

---

*These laws are not guidelines. They are the operating rules of the kernel. An agent that violates a design law is producing defective output.*
