# Domain Bridge: How the Kernel Connects to Domain Practice

> The kernel produces universal structural outputs. Domain practitioners interpret and act on those outputs through the lens of their field. This document explains that connection.

---

## The Key Insight

The kernel is domain-agnostic. It reasons about systems — their structure, coherence, completeness, and evolution — without knowing anything about software, medicine, supply chains, or civil engineering. It works exclusively with abstract primitives: purpose, boundary, input, transformation, output, interface, resource, constraint, feedback loop, failure mode, evolution path.

Domain practitioners are domain-specific. A software architect knows that "interface" means an API contract. A logistics planner knows it means a handoff protocol between carriers. A hospital administrator knows it means an admission intake process. Each expert translates the same structural concept into the vocabulary and practices of their field.

The domain bridge is that translation layer. It connects the kernel's structural language to the practitioner's domain language — without modifying either.

Neither is sufficient alone:
- Kernel without domain knowledge = structurally sound architecture with no grounding in real-world constraints, tools, or practices.
- Domain knowledge without kernel = expert work that may be structurally incomplete, incoherent, or brittle under pressure.

---

## How Kernel Outputs Flow to Domain Practice

### The Universal Structural Output

At every phase, the kernel produces outputs expressed in structural primitives. These outputs are domain-neutral by design. They describe:
- What the system is for (intent, purpose, success criteria)
- What the system consists of (components, boundaries, interfaces)
- How the system behaves (transformations, feedback loops)
- How the system can break (failure modes)
- How the system can change (evolution paths)
- How good the system is (audit vectors with confidence scores)

### The Domain Interpretation Step

A domain practitioner receives these structural outputs and interprets them through their field's vocabulary, tools, standards, and constraints. The interpretation does not change the structural facts — it gives them domain-specific meaning.

The same audit finding — "failure awareness score: 0.42, below threshold 0.65" — is interpreted differently across domains:

| Domain | Interpretation of Low Failure Awareness |
|---|---|
| Software | Missing circuit breakers, no retry logic, unhandled exception paths |
| Healthcare | No escalation protocols, missing triage criteria, undefined handoff failures |
| Logistics | No rerouting procedures, single-carrier dependency, no delay containment plan |
| Organizational Design | No decision escalation path, single points of authority, no conflict resolution process |
| Physical Infrastructure | No load redundancy, single failure point in critical path, no emergency bypass |

The structural diagnosis is the same. The remediation is domain-specific.

---

## Phase-by-Phase: What the Kernel Produces and How Domains Use It

### Phase 1: Intent Compilation

**Kernel produces**: A structured intent object containing purpose statement, identified actors, use cases, system boundary, and domain context signals.

**How domain practitioners use it**:

| Domain | How They Use the Intent Object |
|---|---|
| Software | Validate product requirements, confirm user personas, identify technical constraints not stated in the request |
| Healthcare | Map purpose to clinical workflow, identify patient populations, surface regulatory context (HIPAA, clinical protocols) |
| Logistics | Identify origin-destination pairs, cargo types, time constraints, regulatory crossing requirements |
| Organizational Design | Clarify mission scope, identify stakeholder groups, surface cultural or political constraints |
| Physical Infrastructure | Define service area, identify load populations, surface environmental and regulatory requirements |

---

### Phase 2: Success Model

**Kernel produces**: A success model with measurable dimensions, pass thresholds, and quality dimension priorities.

**How domain practitioners use it**:

| Domain | How They Use the Success Model |
|---|---|
| Software | Set SLAs (latency p99, uptime %, error rate), define user satisfaction metrics, establish security compliance targets |
| Healthcare | Define clinical outcome benchmarks, patient safety thresholds, throughput and wait-time targets |
| Logistics | Set on-time delivery targets, damage rate thresholds, cost-per-unit benchmarks, carbon footprint limits |
| Organizational Design | Define decision speed targets, employee engagement thresholds, cross-team collaboration metrics |
| Physical Infrastructure | Set capacity margins, safety factors, maintenance cycle targets, energy efficiency benchmarks |

---

### Phase 3: Architecture Generation

**Kernel produces**: 2-4 structurally distinct candidate architectures, each representing a different structural thesis for solving the intent.

**How domain practitioners use it**:

| Domain | How They Interpret Candidates |
|---|---|
| Software | Microservices vs. modular monolith vs. event-driven — practitioners evaluate operational feasibility, team fit, technology availability |
| Healthcare | Centralized care model vs. distributed care network vs. hybrid — practitioners evaluate staffing, facility constraints, patient access |
| Logistics | Hub-and-spoke vs. point-to-point vs. relay network — practitioners evaluate fleet utilization, route economics, service level tradeoffs |
| Organizational Design | Hierarchical vs. flat vs. matrix structure — practitioners evaluate communication overhead, accountability clarity, scalability |
| Physical Infrastructure | Single large facility vs. distributed nodes vs. linear network — practitioners evaluate cost, redundancy, geographic coverage |

---

### Phase 4: Candidate Selection

**Kernel produces**: A scored comparison matrix with structural rationale for the recommended candidate or hybridization.

**How domain practitioners use it**:

| Domain | How They Use the Comparison Matrix |
|---|---|
| Software | Validate structural scores with domain-specific feasibility: team skills, vendor availability, operational runbook complexity |
| Healthcare | Cross-reference structural scores with clinical workflow compatibility, staff training requirements, patient impact |
| Logistics | Validate against contractual constraints, carrier relationships, regulatory approval timelines |
| Organizational Design | Check scores against change management capacity, leadership bandwidth, cultural readiness |
| Physical Infrastructure | Validate against permitting timelines, material lead times, contractor availability |

---

### Phase 5: Structural Synthesis

**Kernel produces**: A fully elaborated architecture with all primitives defined — every component, interface, resource, constraint, feedback loop, and failure mode specified.

**How domain practitioners use it**:

| Domain | How They Use the Synthesis |
|---|---|
| Software | Translate into service specifications, API contracts, infrastructure-as-code, data models, deployment plans |
| Healthcare | Translate into care pathway documents, staffing plans, facility layouts, equipment procurement lists, protocol manuals |
| Logistics | Translate into route plans, carrier contracts, warehouse layouts, tracking system requirements, contingency procedures |
| Organizational Design | Translate into org charts, RACI matrices, job descriptions, decision rights documents, governance charters |
| Physical Infrastructure | Translate into engineering drawings, material specifications, construction schedules, safety inspection plans |

---

### Phase 6: Audit

**Kernel produces**: Confidence vectors for all audit dimensions, flagging dimensions that fall below threshold and recommending reroute targets.

**How domain practitioners use it**:

| Domain | How They Use Audit Vectors |
|---|---|
| Software | Translate low-scoring dimensions into specific technical gaps: missing circuit breakers, inadequate test coverage, unversioned APIs |
| Healthcare | Translate low scores into clinical protocol gaps: undefined escalation paths, missing patient safety checkpoints |
| Logistics | Translate low scores into operational gaps: single-carrier risk, missing delay protocols, inadequate tracking |
| Organizational Design | Translate low scores into governance gaps: unclear decision rights, missing conflict resolution paths |
| Physical Infrastructure | Translate low scores into engineering gaps: insufficient redundancy, missing safety margins, inadequate inspection plans |

---

### Phase 7: Delivery

**Kernel produces**: A canonical system package — the complete, structured artifact containing intent, success model, candidates, selection rationale, synthesis, audit results, and evolution ledger.

**How domain practitioners use it**:

| Domain | How They Use the Canonical Package |
|---|---|
| Software | The package becomes the system blueprint: architecture document, API specs, infrastructure plan, onboarding guide |
| Healthcare | The package becomes the care program design: clinical protocols, staffing model, facility plan, training curriculum |
| Logistics | The package becomes the operations manual: route network design, carrier SLAs, contingency playbooks, tracking specs |
| Organizational Design | The package becomes the transformation plan: target org structure, transition roadmap, governance framework, change management plan |
| Physical Infrastructure | The package becomes the project brief: engineering specifications, construction plan, commissioning checklist, operations manual |

---

## Conflict Resolution Across the Bridge

When kernel structural requirements appear to conflict with domain best practices, the following hierarchy applies:

1. **Safety and integrity requirements win unconditionally.** If a structural optimization conflicts with a domain safety requirement, the safety requirement takes precedence. This is non-negotiable in any domain.

2. **Explicit trade-offs are documented.** When structural efficiency conflicts with domain operational practice, the trade-off is recorded in the evolution ledger with rationale. Neither side silently overrides the other.

3. **The kernel defers to domain expertise on domain correctness.** The kernel evaluates the structural quality of a design — coherence, completeness, adaptability — not whether a clinical protocol is medically sound or a logistics route is operationally feasible. Domain practitioners own domain correctness.

4. **Domain practitioners defer to the kernel on structural quality.** If a domain expert proposes a design that is structurally incoherent — components that do not connect, interfaces without contracts, feedback loops that terminate nowhere — the kernel's structural assessment governs.

5. **Unresolvable conflicts escalate.** If structural and domain requirements genuinely cannot be reconciled, the conflict surfaces to a human decision-maker. Neither the kernel nor the domain practitioner resolves it unilaterally.

---

## The Controller's Dual Role

The Controller Architect runs the kernel's pipeline loop as its primary control flow. At each phase, it coordinates domain practitioners to provide domain-specific interpretation and elaboration of structural outputs. The loop does not advance until all domain contributions for the current phase are integrated.

The Controller does not need to understand the domain deeply. It needs to understand the structure of domain contributions: whether they are complete, consistent with accumulated context, and sufficient to ground the kernel's structural outputs in domain reality.

---

## Canonical Package as Universal Artifact

The canonical system package produced by the kernel is the same artifact regardless of domain. Its structure — intent, success model, candidates, evaluation, synthesis, audit, evolution ledger, outputs — is universal. What varies is the content within each field, which is expressed in domain-specific terms.

| Canonical Package Field | What It Contains Across Domains |
|---|---|
| `intent` | Purpose and actors — expressed in domain vocabulary |
| `success_model` | Measurable criteria — using domain-specific metrics and thresholds |
| `candidates` | Structural alternatives — described using domain patterns and tools |
| `evaluation` | Scoring rationale — grounded in domain-specific feasibility |
| `synthesis` | Detailed architecture — elaborated using domain conventions |
| `audit_results` | Confidence vectors — interpreted through domain failure modes |
| `evolution` | Change history — documented using domain change management vocabulary |
| `outputs` | Deliverable artifacts — formatted for domain practitioners |

The package is the bridge made artifact. It contains both the structural logic (from the kernel) and the domain expression (from practitioners) in a single coherent document.

---

## Creating a Domain-Specific Bridge

To formalize how the kernel connects to a new domain:

1. **Map all 11 system primitives** to domain-specific concepts. Every primitive must have at least one concrete, actionable translation in the target domain.

2. **Identify the practitioner roles** who interpret and act on kernel outputs at each phase. These are the domain equivalents of the kernel's specialist agents.

3. **Define domain-specific audit evidence**. The 8 universal audit dimensions apply in every domain, but the evidence and measurement methods are domain-specific. Define what "coherent" and "complete" mean in the target domain.

4. **Document characteristic failure modes** for the domain. These ground the kernel's generic failure mode reasoning in domain-specific risk patterns.

5. **Define the deliverable format** for the canonical package in this domain — what documents, artifacts, or specifications the package translates into for downstream practitioners.

6. **Register the bridge** in the kernel manifest so the Controller can load it based on the domain classification in the intent object.

---

*Domain Bridge — Kernel Integration Layer v1.1*
*This document replaces the previous software-specific kernel_os_bridge.md.*
