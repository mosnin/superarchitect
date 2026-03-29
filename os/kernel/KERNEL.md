# SuperArchitect Kernel -- Architecture Reasoning Engine

> The kernel is not a blueprint writer. It is not a checklist. It is a systems compiler that translates intent into system geometry.

---

## What is the Kernel?

The kernel is a domain-agnostic architecture reasoning engine. It is the structural brain of the entire system.

The kernel carries structural cognition. It knows how to reason about systems at the level of shape, tradeoff, measurement, and evolution -- regardless of whether the system is a logistics routing network, a hospital management platform, a financial settlement engine, or a distributed data processing system.

The kernel does not generate code. It does not write infrastructure configs. It does not design UIs. It generates the system geometry that all of those activities build from. Everything downstream -- every interface contract, every data model, every deployment topology -- is an expression of kernel output.

### What the kernel is

1. A staged architecture compiler with selective recursion
2. A search engine over structural design space
3. A measurable audit system with machine-readable quality vectors
4. A routing engine that sends weak dimensions back to the shallowest fixable stage
5. A canonical state manager that maintains one YAML package as the single source of truth

### What the kernel is NOT

1. A domain knowledge library
2. A generic checklist collection
3. A one-pass prompt template
4. A static blueprint generator
5. A software-only framework

---

## Core Thesis

The kernel separates STRUCTURAL cognition from DOMAIN cognition.

Structural cognition is the ability to reason about systems using universal primitives: purpose, boundary, inputs, transformations, outputs, interfaces, resources, constraints, feedback loops, failure modes, and evolution paths. These primitives apply to any system in any domain.

Domain cognition is the ability to reason about specific technical or business concerns: technology selection, compliance requirements, processing architectures, data modeling strategies, deployment topologies. This expertise is injected into the kernel pipeline at specific phases.

The kernel handles structural cognition. Domain expertise is applied at the points where structural decisions require it. This separation is what prevents architecture reasoning from degenerating into a bag of ad-hoc heuristics.

Every domain differs in content. Most domains share structural laws. The kernel operates on those shared laws.

---

## The 7-Phase Pipeline

The kernel processes every architecture request through a staged pipeline. Each phase has defined inputs, outputs, responsibilities, and fail conditions. Phases are not cosmetic labels -- they enforce real cognitive separation between exploration, evaluation, synthesis, and audit.

### Phase 1: Intent Compilation

Convert raw request into structured intent object. Extract objectives, constraints, ambiguities, consequence level, and output requirements. The messy human request becomes a machine-readable architecture input.

- **Kernel agents activated**: Intent Analyst (primary), Controller Architect (oversight)
- **Output**: Intent object, ambiguity map, consequence rating

### Phase 2: Success Model Generation

Derive what "world class" means for THIS specific project. Merge universal backbone dimensions (coherence, completeness, consistency, adaptability, efficiency, failure awareness, legibility, implementability) with project-specific dimensions derived from intent. Define tradeoff ordering and failure conditions.

- **Kernel agents activated**: Success Model Architect (primary), Controller Architect (threshold validation)
- **Output**: Success model with scored dimensions, thresholds, tradeoff hierarchy

### Phase 3: Architecture Search

Generate 3+ genuinely different structural theses. Not cosmetic variations. Each candidate must represent a fundamentally different tradeoff stance: modularity-first, simplicity-first, robustness-first, adaptability-first, performance-first.

- **Kernel agents activated**: Search Architect (primary), Controller Architect (diversity check)
- **Output**: Candidate set with encoded assumptions, advantages, liabilities

### Phase 4: Comparative Reasoning

Score and compare candidates against the success model. Produce a comparative matrix. Rank candidates. Determine whether to select one winner or hybridize the best traits from multiple. Document WHY the chosen direction won.

- **Kernel agents activated**: Comparative Reasoner (primary), Failure Mode Architect (adversarial review), Optimization Architect (efficiency review), Controller Architect (selection authority)
- **Output**: Comparative matrix, winner/hybrid decision, rationale

### Phase 5: Structural Synthesis

Turn the winning thesis into complete system geometry. Define subsystems, interfaces, flows, dependencies, control points, feedback loops, failure containment, and evolution paths. Every system primitive must be represented.

- **Kernel agents activated**: Synthesis Architect (primary), Failure Mode Architect (failure containment), Optimization Architect (simplification), Controller Architect (coherence check)
- **Output**: Synthesized architecture object with all primitives

### Phase 6: Audit and Routing

Measure quality with machine-readable vectors. For each dimension emit score, evidence strength, uncertainty, delta from previous pass, and status. If dimensions fall below threshold, route back to the shallowest stage that can fix the defect.

- **Kernel agents activated**: Audit Architect (primary), Mutation Architect (on reroute), Controller Architect (routing authority)
- **Output**: Audit vector, routing decision, evolution ledger entry

### Phase 7: Packaging

Freeze canonical package. Generate human-readable blueprint markdown derived FROM the package. Produce handoff manifest for downstream consumers. Persist audit history.

- **Kernel agents activated**: Packaging Architect (primary), Controller Architect (final sign-off)
- **Output**: final_system_package.yaml, system_blueprint.md, handoff_manifest.yaml

---

## Two Kinds of Iteration

The kernel iterates, but not blindly. There are exactly two iteration modes, and they serve different purposes.

### Exploratory Iteration

Used BEFORE a winning architecture exists. Occurs between Phases 3 and 4.

Purpose: broaden the search space. Generate more candidates, explore different thesis axes, avoid premature anchoring on a single structural approach. The kernel stays in exploratory mode when candidate diversity is insufficient or when the comparative matrix shows no clear winner.

Rules:
- Never collapse to one candidate too early
- Each iteration must introduce genuinely new structural ideas
- Exit when meaningful variation is achieved and a winner can be selected

### Corrective Iteration

Used AFTER a winner exists. Occurs between Phases 5 and 6.

Purpose: targeted repair. When audit identifies weak dimensions, the kernel routes back to the specific stage that can fix the defect. This is surgical correction, not wholesale redesign.

Rules:
- Never discard the entire architecture -- fix the weak part
- Route to the SHALLOWEST stage that can resolve the defect
- Every reroute appends an entry to the evolution ledger
- Exit when all dimensions pass threshold or maximum iterations reached

---

## Kernel Agents

The kernel operates through 11 specialized cognitive agents. Each agent owns a specific type of reasoning. Agents are not personas -- they are cognitive functions.

| # | Agent | Cognitive Function |
|---|-------|--------------------|
| 1 | **Controller Architect** | Coherence, stage transitions, package authority, reroute decisions |
| 2 | **Intent Analyst** | Objective extraction, ambiguity mapping, consequence estimation |
| 3 | **Success Model Architect** | Quality dimension derivation, threshold setting, tradeoff ordering |
| 4 | **Search Architect** | Candidate generation, thesis variation, structural exploration |
| 5 | **Comparative Reasoner** | Scoring, ranking, tradeoff analysis, winner/hybrid selection |
| 6 | **Synthesis Architect** | System geometry construction, subsystem/interface/flow definition |
| 7 | **Failure Mode Architect** | Adversarial pressure testing, breakdown/drift/overload analysis |
| 8 | **Optimization Architect** | Elegance, efficiency, simplification, over-engineering detection |
| 9 | **Audit Architect** | Measurable vector production, reroute targeting, evidence grading |
| 10 | **Mutation Architect** | Focused corrective changes during reroute, surgical fixes |
| 11 | **Packaging Architect** | Package normalization, blueprint generation, handoff preparation |

Agent definitions: `os/kernel/agents/`

---

## Authority and Coherence

The Controller Architect is the single coherence owner across all 7 phases. It makes all routing decisions, owns the canonical package, manages stage transitions, and is the final authority on whether a phase output is sufficient to proceed.

No phase advances without the Controller Architect's approval. No reroute happens without the Controller Architect selecting the target stage. No package is frozen without the Controller Architect's sign-off.

All other kernel agents report to and operate under the direction of the Controller Architect.

---

## The Canonical System Package

Every project processed by the kernel produces exactly one canonical system package. This is a YAML object that serves as the single source of truth for the entire architecture lifecycle.

All phases read from and write to the package. No phase invents hidden state outside the package. If a phase produces intermediate artifacts, it writes a reference back into the package manifest.

### Package sections

```yaml
metadata:        # package_id, project_id, version, timestamps
intent:          # compiled intent object from Phase 1
success_model:   # quality dimensions, thresholds, tradeoffs from Phase 2
candidates:      # architecture candidates from Phase 3
selection:       # chosen candidate and rationale from Phase 4
synthesis:       # full system geometry from Phase 5
audit:           # current and historical audit vectors from Phase 6
routing_state:   # current phase, reroute target, status
evolution:       # immutable ledger of all reroutes and mutations
outputs:         # paths to blueprint, manifest, artifacts
handoff:         # downstream targets and integration notes
```

Package template: `os/kernel/templates/final_package_template.yaml`

---

## Audit Vector Mechanics

Each audit vector entry carries five fields per dimension:

- **score**: Numeric quality rating (0.0 -- 1.0) against the success model threshold for this dimension
- **evidence**: Evidence strength rating (none / weak / moderate / strong) -- how well the score claim is supported
- **uncertainty**: Uncertainty level (low / medium / high) -- how confident the rating is given available information
- **delta**: Change from the previous audit pass (positive = improvement, negative = regression, null = first pass)
- **status**: Pass / Fail / Watch -- whether the dimension meets threshold, fails it, or is borderline

### Universal Backbone Dimensions

Every project is audited against these 8 structural dimensions regardless of domain:

1. **Coherence** -- all parts of the system work toward the same purpose without contradiction
2. **Completeness** -- all necessary subsystems, interfaces, and flows are present
3. **Consistency** -- naming, patterns, and structural conventions are uniform throughout
4. **Adaptability** -- the system has explicit mechanisms for evolution without full redesign
5. **Efficiency** -- no unnecessary complexity, redundancy, or resource waste
6. **Failure Awareness** -- failure modes are identified, contained, and handled explicitly
7. **Legibility** -- the system's intent and structure can be understood by a new reader
8. **Implementability** -- the design can be realized by a competent team without unresolvable ambiguity

Project-specific dimensions are added by the Success Model Architect during Phase 2, layered on top of these 8.

---

## Targeted Rerouting Logic

When Phase 6 audit identifies dimensions below threshold, the Controller Architect selects a reroute target using these rules:

| Failing Dimension | Likely Root Cause | Reroute Target |
|-------------------|-------------------|----------------|
| Coherence | Structural contradiction between subsystems | Phase 5 (Synthesis) |
| Completeness | Missing subsystems or interfaces | Phase 5 (Synthesis) |
| Consistency | Pattern drift or naming inconsistency | Phase 5 (Synthesis) |
| Adaptability | No evolution paths defined | Phase 5 (Synthesis) |
| Failure Awareness | Failure modes unanalyzed | Phase 5 (Synthesis) via Failure Mode Architect |
| Efficiency | Over-engineering or redundancy | Phase 5 (Synthesis) via Optimization Architect |
| Legibility | Architectural intent unclear | Phase 5 (Synthesis) |
| Implementability | Unresolvable ambiguity | Phase 1 (Intent) or Phase 5 (Synthesis) |
| Candidate diversity | All candidates share same structural thesis | Phase 3 (Search) |
| Selection rationale | Winner not clearly superior | Phase 4 (Comparative Reasoning) |

The Mutation Architect performs the targeted fix at the reroute stage. Every reroute is recorded in the evolution ledger with: timestamp, failing dimension(s), reroute target, agent responsible, and change summary.

---

## Evolution Ledger

The evolution ledger is an immutable append-only log within the canonical package. It records every corrective iteration the system underwent from initial synthesis to final freeze.

Each ledger entry contains:
- **entry_id**: Sequential identifier
- **timestamp**: When the reroute occurred
- **trigger**: Which dimension(s) failed audit and at what score
- **reroute_target**: Which phase was re-entered
- **agent**: Which kernel agent performed the mutation
- **change_summary**: What was structurally modified and why
- **pre_scores**: Audit scores before the mutation
- **post_scores**: Audit scores after the mutation (populated on next audit pass)

The ledger is never edited or deleted. It provides a complete causal history of how the architecture reached its final form.

---

## Runtime Modes

The kernel adapts its behavior based on project characteristics and audit state.

### Standard
Balanced exploration and correction. Default mode. Suitable for most projects.

### Search Heavy
More candidate architectures generated. Deeper comparison. Slower convergence. Used when the problem space is novel or when early candidates show insufficient diversity.

### Conservative
Higher audit thresholds. Stronger evidence requirements. More frequent escalation. Used for high-consequence projects where failure cost is extreme (safety-critical systems, irreversible large-scale processes).

### Overdrive
Activated when dimensions fall below critical adaptive thresholds during audit. Spawns focused specialist passes on weak dimensions. This is not a general mode -- it is triggered by specific audit failures and deactivated when those dimensions recover.

Mode selection is made by the Controller Architect based on consequence level, uncertainty, audit weakness, and exploration needs.

---

## Escalation Policy

The kernel defaults to full autonomy. It makes all structural decisions without human confirmation.

Escalation occurs ONLY when ALL of the following conditions hold:
1. Uncertainty is materially high (evidence is insufficient to distinguish options)
2. Consequence is high (wrong decision causes significant damage)
3. The decision depends on human preference or authority (not technical fitness)

Examples of justified escalation:
- Top two candidates score within 5% and tradeoffs are strategic (cost vs. speed)
- Intent contains genuine ambiguity that cannot be resolved from context
- Regulatory or legal constraints require human sign-off

Examples of unjustified escalation:
- Choosing between two structural approaches when one clearly scores higher
- Selecting a system topology when requirements are unambiguous
- Making any decision where the kernel has sufficient evidence to proceed

---

## Anti-Slop Criteria

The kernel rejects its own outputs when they exhibit any of the following defects:

1. **Vague**: Architecture descriptions that could apply to any system
2. **Overabstract**: High-level concepts without structural specifics
3. **Checklist-heavy without structure**: Lists of requirements without system geometry
4. **Internally contradictory**: Components that conflict with each other or with stated goals
5. **Underaudited**: Claims without measurable evidence or uncertainty ratings
6. **Overconfident without evidence**: High scores with low evidence strength
7. **Too rigid**: Architectures with no evolution paths or adaptation mechanisms

These criteria are enforced at Phase 6 (Audit) and serve as automatic reroute triggers. An architecture that passes dimensional scores but fails anti-slop criteria is still routed back for correction.

---

## How to Read This Kernel

Agents loading the SuperArchitect OS should read kernel files in this order:

1. **This file** (`os/kernel/KERNEL.md`) -- understand the kernel's identity, pipeline, and agent structure
2. **Principles** (`os/kernel/principles/`) -- internalize system primitives, design laws, and world-class standard
3. **Phase specs** (`os/kernel/phases/01-07`) -- understand each pipeline stage in depth
4. **Agent specs** (`os/kernel/agents/`) -- understand each kernel agent's cognitive function
5. **Templates** (referenced within phase specs) -- understand the data structures

Do not read selectively. The kernel is a coherent system. Partial reading produces partial understanding, which produces defective architecture.

---

## File Map

```
os/kernel/
  KERNEL.md                              # This file -- kernel identity and architecture
  phases/
    01_intent_compilation.md             # Phase 1 spec
    02_success_model.md                  # Phase 2 spec
    03_architecture_search.md            # Phase 3 spec
    04_comparative_reasoning.md          # Phase 4 spec
    05_structural_synthesis.md           # Phase 5 spec
    06_audit_and_routing.md              # Phase 6 spec
    07_packaging.md                      # Phase 7 spec
  agents/
    controller_architect.md              # Agent 1 -- coherence and orchestration
    intent_analyst.md                    # Agent 2 -- intent extraction
    success_model_architect.md           # Agent 3 -- quality dimension derivation
    search_architect.md                  # Agent 4 -- candidate generation
    comparative_reasoner.md              # Agent 5 -- scoring and selection
    synthesis_architect.md               # Agent 6 -- system geometry
    failure_mode_architect.md            # Agent 7 -- adversarial testing
    optimization_architect.md            # Agent 8 -- elegance and efficiency
    audit_architect.md                   # Agent 9 -- measurable audit
    mutation_architect.md                # Agent 10 -- corrective changes
    packaging_architect.md               # Agent 11 -- final packaging
  principles/
    system_primitives.md                 # 11 universal system primitives
    design_laws.md                       # 10 kernel design laws
    world_class_standard.md              # Definition of world-class
```

---

*SuperArchitect Kernel v1.0 -- The reasoning engine of the OS*
*Structural cognition, separated from domain cognition, measured at every stage.*
