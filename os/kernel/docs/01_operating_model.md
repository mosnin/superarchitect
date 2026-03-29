# Kernel Operating Model

> The kernel is a staged system compiler with selective recursion.
> It advances linearly through 7 phases, rerouting backward only when the audit layer identifies a dimensional deficiency.

---

## The 7-Phase Pipeline

The kernel processes every build request through a fixed sequence of seven phases. Each phase has a single responsibility, defined inputs, defined outputs, and a quality gate that must be satisfied before advancing.

```
Phase 1: Intent Compilation
    |
Phase 2: Success Model Generation
    |
Phase 3: Architecture Search (candidate generation)
    |
Phase 4: Comparative Reasoning (scoring + selection)
    |
Phase 5: Structural Synthesis (full design assembly)
    |
Phase 6: Audit (dimensional scoring + reroute decision)
    |  \_____ reroute to shallowest deficient phase
    |
Phase 7: Packaging (canonical system package + derived artifacts)
```

### Phase 1 — Intent Compilation

**Input**: Raw build request (natural language description from human operator)
**Output**: Structured intent object (YAML)
**Responsibility**: Parse the request into a formal intent with objective, desired outcomes, known constraints, consequence level, and ambiguity notes.

The intent compiler does not invent requirements. It extracts what is stated, flags what is ambiguous, and assigns a consequence level (low / medium / high / critical) that governs how conservative the kernel's decisions will be throughout the build.

**Quality gate**: Intent object has non-empty objective, at least one desired outcome, and a consequence level assigned.

### Phase 2 — Success Model Generation

**Input**: Structured intent object
**Output**: Success model (YAML) with scored dimensions and thresholds
**Responsibility**: Define what "good enough" means for this specific system. Combine the 8 universal backbone dimensions with project-specific dimensions derived from the intent.

The success model is not aspirational. Every dimension has a **threshold** — the minimum acceptable score. Dimensions also have a **tradeoff priority** that governs which dimensions yield when they conflict.

**Quality gate**: All backbone dimensions present. At least one project-specific dimension defined. Tradeoff priority explicitly ordered.

### Phase 3 — Architecture Search

**Input**: Intent object + success model
**Output**: 2-4 candidate architectures (YAML)
**Responsibility**: Generate meaningfully different architectural approaches to the stated problem. Each candidate must include a thesis, key assumptions, advantages, liabilities, and subsystem breakdown.

Candidates must be **genuinely distinct** — not minor variations of the same idea. If the kernel generates three microservice variants, it has failed the search phase. Good search produces candidates that represent different **structural bets** (e.g., monolith vs. microservices vs. edge-first hybrid).

**Quality gate**: At least 2 candidates generated. Each candidate has a distinct thesis. No two candidates share the same primary structural bet.

### Phase 4 — Comparative Reasoning

**Input**: Candidate architectures + success model
**Output**: Comparative matrix with dimensional scores, winner selection, and hybridization decisions
**Responsibility**: Score every candidate against every success model dimension. Select a winner or synthesize a hybrid from the strongest elements of multiple candidates.

This phase produces the most important artifact in the kernel pipeline: the **comparative matrix**. This matrix is a full cross-product of candidates and dimensions, with scores (0.0-1.0), evidence, and confidence levels for each cell.

Hybridization is permitted and encouraged when different candidates excel on different dimensions. The kernel documents exactly which elements are taken from which candidate and why.

**Quality gate**: All candidates scored on all dimensions. Winner or hybrid explicitly selected with documented rationale. No dimension left unscored.

### Phase 5 — Structural Synthesis

**Input**: Winning architecture (or hybrid) + success model + intent
**Output**: Complete structural design (YAML) with all 11 system primitives populated
**Responsibility**: Expand the winning candidate into a full system design. Define every subsystem, every interface, every data flow, every control point, every feedback loop, every failure containment mechanism, and every evolution path.

This is where architecture becomes engineering. The synthesis phase produces the level of detail that an implementation team needs to begin building. Interfaces have protocols and payload schemas. Flows have sequence descriptions. Failure modes have specific containment strategies.

**Quality gate**: All 11 system primitives populated. Every interface has a defined protocol. Every subsystem has at least one failure mode analyzed.

### Phase 6 — Audit

**Input**: Synthesized design + success model
**Output**: Audit vector (dimensional scores + evidence + reroute decision)
**Responsibility**: Score the synthesized design against every success model dimension. Identify any dimension below threshold. Decide whether to advance or reroute.

The audit phase is the kernel's **optimization engine**. It does not just pass/fail — it produces a full confidence vector, identifies the weakest dimension, and if rerouting is needed, targets the **shallowest phase** capable of fixing the deficiency.

If all dimensions meet their thresholds, the kernel advances to Phase 7. If any dimension fails, the kernel reroutes and the evolution ledger records the iteration.

**Quality gate**: All dimensions scored. Reroute decision explicitly made. If rerouting, target phase and mutation strategy documented.

### Phase 7 — Packaging

**Input**: Audited design (all dimensions passing)
**Output**: Canonical system package (YAML) + derived artifacts (blueprint, handoff manifest, audit report)
**Responsibility**: Assemble the final canonical system package. Generate all derived artifacts. Prepare the handoff manifest for implementation dispatch.

**Quality gate**: Package validates against the canonical schema. All 11 sections present. Blueprint and handoff manifest generated.

---

## Why Stages, Not One Freeform Loop

A common alternative to staged compilation is a single freeform reasoning loop: "think about the system until it's good enough, then output it." This approach has four structural weaknesses that stages prevent:

### 1. Preserves Real Exploration

In a freeform loop, the agent tends to commit to the first plausible design and refine it. Stages force the kernel to generate **multiple candidates** (Phase 3) before selecting one (Phase 4). This preserves genuine exploration of the design space rather than hill-climbing on the first idea.

### 2. Explicit Tradeoff Reasoning

When design and evaluation happen simultaneously, tradeoffs are made implicitly. The staged pipeline separates **generation** (Phase 3) from **evaluation** (Phase 4), ensuring that every tradeoff is explicitly scored and documented in the comparative matrix.

### 3. Local Correction

When a freeform loop discovers a problem, it tends to patch locally — fixing the symptom without addressing the root cause. The kernel's rerouting model sends deficiencies back to the **shallowest phase** that can fix them, ensuring corrections happen at the right level of abstraction.

### 4. Measurable Improvement

Each pass through the audit phase produces a dimensional score vector. Successive iterations must show **monotonic improvement** on the failing dimension. If an iteration does not improve the score, the kernel escalates rather than looping indefinitely.

---

## Two Kinds of Iteration

The kernel distinguishes between two fundamentally different types of iteration:

### Exploratory Iteration (Before Winner Selection)

Occurs in Phases 1-3. The kernel may regenerate candidates if the initial search produces insufficient diversity or if the intent compilation reveals new constraints. Exploratory iteration **expands the search space**.

- Triggered by: insufficient candidate diversity, ambiguous intent requiring re-parsing
- Bounded by: maximum 2 exploratory iterations per phase
- Direction: forward-looking (generate more options)

### Corrective Iteration (After Winner Selection)

Occurs in Phases 4-7. The audit layer identifies a dimensional deficiency and reroutes to an earlier phase. Corrective iteration **narrows toward a specific improvement**.

- Triggered by: audit dimension below threshold
- Bounded by: maximum 3 corrective iterations per build (then escalate)
- Direction: backward-looking (fix a specific weakness)

This distinction matters because the two types require different strategies. Exploratory iteration should be creative and divergent. Corrective iteration should be surgical and convergent.

---

## The Routing Rule

When the audit layer identifies a failing dimension, the kernel must decide which phase to revisit. The rule is:

> **Return to the shallowest phase that can fix the defect.**

"Shallowest" means earliest in the pipeline. The logic:

| Deficiency Type | Reroute Target | Rationale |
|----------------|----------------|-----------|
| Wrong objective or missing constraint | Phase 1 (Intent) | The problem is in what we are building |
| Wrong success criteria or thresholds | Phase 2 (Success Model) | The problem is in how we define success |
| No candidate addresses the weak dimension | Phase 3 (Search) | The design space was not explored enough |
| Wrong candidate selected | Phase 4 (Comparative) | The selection logic was flawed |
| Design is incomplete or has structural gaps | Phase 5 (Synthesis) | The design needs more detail or restructuring |

Most reroutes target Phase 5 (structural synthesis) — the design is directionally correct but needs refinement. Reroutes to Phase 1 or 2 are rare and indicate a fundamental misunderstanding of the problem.

---

## Decision Styles

The kernel adapts its decision-making aggressiveness based on the consequence level declared in the intent:

### Autonomous Progression (consequence: low)

The kernel advances through phases with minimal deliberation. Candidate search generates 2 options. Audit thresholds are relaxed (0.6 minimum). Single iteration usually sufficient.

**Appropriate for**: Internal tools, prototypes, experimental features.

### Conditional Escalation (consequence: medium)

Standard mode. 3 candidates generated. Audit thresholds at 0.7. Up to 2 corrective iterations permitted. Escalate to human if second iteration does not improve scores.

**Appropriate for**: Customer-facing features, standard SaaS products, team infrastructure.

### Conservative Mode (consequence: high)

Extra scrutiny at every phase. 3-4 candidates generated. Audit thresholds at 0.75. Security and reliability dimensions weighted 1.5x. Up to 3 corrective iterations. Mandatory human review of Phase 4 comparative matrix before proceeding.

**Appropriate for**: Financial systems, healthcare, compliance-critical infrastructure, systems with data loss consequences.

### Aggressive Search (consequence: critical)

Maximum exploration. 4 candidates generated with mandatory diversity constraints. Audit thresholds at 0.8. All dimensions must pass. No automatic advancement from Phase 4 — human must approve winner selection. Full threat model required in Phase 5.

**Appropriate for**: Systems where failure has safety, legal, or catastrophic business consequences.

---

## Integration with OS Teams

The kernel does not operate in isolation. Each kernel phase dispatches work to specialist OS teams:

| Kernel Phase | Primary Team | Supporting Teams |
|-------------|-------------|-----------------|
| Phase 1: Intent Compilation | Team 9 (Product) | Team 10 (Research) |
| Phase 2: Success Model | Team 2 (Architecture) | Team 7 (Security), Team 8 (QA) |
| Phase 3: Architecture Search | Team 2 (Architecture) | Team 10 (Research) |
| Phase 4: Comparative Reasoning | Team 2 (Architecture) | Team 7 (Security), Team 5 (Data) |
| Phase 5: Structural Synthesis | Team 2 (Architecture) | Team 3 (Backend), Team 4 (Frontend), Team 6 (DevOps) |
| Phase 6: Audit | Team 8 (QA) | Team 7 (Security) |
| Phase 7: Packaging | Team 1 (Foundation) | Commander |

The Commander orchestrates this dispatch. Teams receive their instructions through the protocol defined in `os/commander/protocols.md` and return structured results that the kernel integrates into the canonical package.

---

## Phase Timing

Typical phase durations for a medium-complexity system (consequence: medium):

| Phase | Typical Duration | Notes |
|-------|-----------------|-------|
| Phase 1 | 5-10 minutes | Fastest phase; mostly parsing |
| Phase 2 | 10-15 minutes | Requires domain analysis |
| Phase 3 | 15-25 minutes | Most creative phase |
| Phase 4 | 10-20 minutes | Scoring is systematic |
| Phase 5 | 20-40 minutes | Most detailed phase |
| Phase 6 | 10-15 minutes | Systematic evaluation |
| Phase 7 | 5-10 minutes | Mostly assembly |

Total first-pass time: 75-135 minutes. Each corrective iteration adds 15-45 minutes depending on reroute depth.

---

*Kernel Operating Model v1.0 — SuperArchitect OS*
