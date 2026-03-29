# Kernel-OS Integration Bridge

> The kernel provides structural reasoning. The OS provides domain expertise. Together they produce architectures that are both structurally sound and domain-excellent.

---

## The Key Insight

The kernel is domain-agnostic. It knows about systems — their structure, coherence, completeness, and evolution — but it knows nothing about software, databases, APIs, or Kubernetes. It works with abstract primitives: purpose, boundary, input, transformation, output, interface, resource, constraint, feedback, failure mode, evolution path.

The OS teams are domain-specific. They know software deeply — they know that "interface" means REST API or gRPC, that "resource" means PostgreSQL or Redis, that "failure mode" means timeout or deadlock. But they may not naturally reason about structural coherence or systematic completeness.

The bridge connects these two worlds. Kernel phases provide the WHAT (what structural questions must be answered). OS teams provide the HOW (how those questions are answered in the software domain).

Neither is sufficient alone:
- Kernel without OS teams = structurally perfect architecture with no domain grounding.
- OS teams without kernel = domain-expert work that may be structurally incomplete or incoherent.

---

## Phase-to-Team Dispatch Mapping

This table defines which OS teams are activated at each kernel phase and what role they play.

### Phase 1: Intent Compilation

**Kernel asks**: What is this system's purpose? What problem does it solve? For whom? What are the boundaries?

**OS teams dispatched**:
| Team | Role in This Phase |
|---|---|
| Team 9 (Product) | Translate the build request into structured requirements. Identify users, use cases, and business objectives. Define the product boundary. |
| Team 10 (Research) | Research the problem domain. Identify prior art, competitive systems, and reference architectures. Surface constraints the requester may not have mentioned. |

**Output flows to kernel**: Structured intent document with purpose, users, use cases, boundaries, and domain context.

### Phase 2: Success Model

**Kernel asks**: How do we measure success? What are the thresholds? What quality dimensions matter most?

**OS teams dispatched**:
| Team | Role in This Phase |
|---|---|
| Team 9 (Product) | Define user-facing success metrics (performance, reliability, usability targets). Prioritize quality dimensions from the user's perspective. |
| Team 10 (Research) | Provide industry benchmarks and comparable system metrics. Ground the success criteria in reality. |

**Output flows to kernel**: Success model with measurable criteria, thresholds, and quality dimension priorities that feed into `threshold_logic.md`.

### Phase 3: Architecture Search

**Kernel asks**: What are 3+ structurally different approaches to building this system?

**OS teams dispatched**:
| Team | Role in This Phase |
|---|---|
| Team 2 (Architecture) | Generate candidate architectures using domain patterns (microservices, modular monolith, event-driven, serverless, etc.). Each candidate must be structurally distinct. Apply patterns from `os/patterns/architectural/`. |

**Output flows to kernel**: 3+ candidate architecture descriptions, each with component inventory, interface definitions, technology choices, and trade-off analysis.

### Phase 4: Comparative Reasoning

**Kernel asks**: Which candidate best satisfies the success model? Score each against the 8 audit dimensions.

**OS teams dispatched**:
| Team | Role in This Phase |
|---|---|
| Team 2 (Architecture) | Evaluate architectural fitness of each candidate. Score coherence, completeness, adaptability, efficiency. |
| Team 7 (Security) | Evaluate security posture of each candidate. Score failure awareness from a security perspective. Identify attack surface differences. |
| Team 5 (Data) | Evaluate data architecture of each candidate. Score data integrity, consistency, and scalability. |

**Output flows to kernel**: Per-candidate score matrix with domain-specific justifications. Kernel combines with structural scoring to select the winner.

### Phase 5: Structural Synthesis

**Kernel asks**: Elaborate the selected candidate into a complete, detailed architecture with all primitives defined.

**OS teams dispatched**:
| Team | Role in This Phase |
|---|---|
| Team 2 (Architecture) | Produce the detailed architecture: service decomposition, component design, data flow diagrams, integration patterns. |
| Team 5 (Data) | Design the data model, storage strategy, data flow, and data quality mechanisms. |
| Team 7 (Security) | Define authentication, authorization, encryption, secret management, and threat model. |
| Team 6 (DevOps) | Define infrastructure architecture, deployment strategy, CI/CD pipeline, monitoring stack. |
| Team 4 (Frontend) | Define the frontend architecture, component hierarchy, state management, and API consumption patterns (if applicable). |

**Output flows to kernel**: Complete architecture document covering all system primitives. This is the most heavily staffed phase.

### Phase 6: Audit

**Kernel asks**: Does this architecture pass all 8 audit dimensions?

**OS teams dispatched**:
| Team | Role in This Phase |
|---|---|
| Team 8 (QA) | Evaluate testability, test strategy feasibility, and coverage potential. Contribute to completeness and implementability scores. |
| Team 7 (Security) | Security-focused audit. Contribute to failure awareness, internal consistency (security assumptions), and completeness (security primitives). |
| Team 2 (Architecture) | Architectural review. Contribute to coherence, efficiency, adaptability, and legibility scores. |

**Output flows to kernel**: Domain-grounded confidence vectors for all 8 dimensions. The kernel aggregates these with its structural assessment to produce the final audit result.

### Phase 7: Packaging

**Kernel asks**: Compile everything into the canonical system package.

**OS teams dispatched**:
| Team | Role in This Phase |
|---|---|
| Commander | Orchestrate the final assembly. Ensure all artifacts are present and consistent. |
| Team 4 (Frontend) | Produce UI/UX documentation and design system artifacts (if applicable). |

**Output flows to kernel**: Complete canonical system package ready for delivery.

---

## Message Protocol

Communication between kernel phases and OS teams extends the protocol defined in `os/commander/protocols.md`.

### Kernel-to-Team Dispatch Message

```json
{
  "type": "KERNEL_DISPATCH",
  "from": "kernel.controller",
  "to": "os.team.<team_number>",
  "phase": "PHASE_3",
  "iteration": 2,
  "task": {
    "description": "Generate 3 structurally distinct candidate architectures for the system described in the intent document.",
    "constraints": [
      "Each candidate must use a different architectural style.",
      "Each candidate must address all system primitives identified in Phase 1.",
      "Candidates must be feasible with the technology constraints from the success model."
    ],
    "inputs": {
      "intent_document": "...",
      "success_model": "...",
      "reroute_context": null
    },
    "expected_output_format": "candidate_architecture_set",
    "deadline": "phase_budget"
  }
}
```

### Team-to-Kernel Result Message

```json
{
  "type": "KERNEL_RESULT",
  "from": "os.team.2",
  "to": "kernel.controller",
  "phase": "PHASE_3",
  "iteration": 2,
  "result": {
    "status": "complete",
    "artifacts": [ ... ],
    "confidence": 0.85,
    "unresolved_questions": [],
    "handoff_notes": "Candidate B uses an event-driven pattern that requires the team to evaluate message broker options in Phase 5."
  }
}
```

### Reroute Context Message

When a team is re-activated due to a reroute, it receives additional context:

```json
{
  "type": "KERNEL_DISPATCH",
  "reroute": true,
  "reroute_context": {
    "failed_dimensions": ["failure_awareness"],
    "specific_feedback": "The architecture lacks circuit breakers between the payment and order services. The cascading failure path is uncontained.",
    "previous_attempt_summary": "First attempt defined failure modes but did not add containment. Second attempt must add structural containment.",
    "evolution_ledger_excerpt": [ ... ]
  }
}
```

---

## Context Passing

### What Flows from Kernel to Teams

| Context | Description | Available From |
|---|---|---|
| Build request | Original human request | Phase 1 onward |
| Intent document | Structured purpose, users, boundaries | Phase 2 onward |
| Success model | Measurable criteria and thresholds | Phase 3 onward |
| Candidate architectures | All generated candidates | Phase 4 onward |
| Selected candidate | The winning architecture | Phase 5 onward |
| Detailed architecture | Full structural synthesis output | Phase 6 onward |
| Audit results | Confidence vectors for all dimensions | Phase 7 (and reroute targets) |
| Evolution ledger | History of reroutes and mutations | All rerouted phases |
| Reroute instructions | What to fix and why | Rerouted phases only |

### What Flows from Teams to Kernel

| Context | Description | Produced By |
|---|---|---|
| Phase artifacts | The primary output of the phase (designs, evaluations, audits) | All teams |
| Confidence assessment | How confident the team is in its output | All teams |
| Unresolved questions | Issues the team identified but could not resolve | All teams |
| Handoff notes | Information the next phase needs to know | All teams |
| Domain warnings | Domain-specific risks or concerns | Specialist teams (Security, Data) |

---

## Conflict Resolution

When kernel structural requirements conflict with domain best practices:

### Resolution Hierarchy

1. **Safety and security requirements win**. If the kernel's structural optimization conflicts with a security requirement, security wins. Non-negotiable.

2. **Explicit trade-offs are documented**. If efficiency (kernel) conflicts with operational best practice (OS team), the trade-off is documented in the evolution ledger and the decision record.

3. **The kernel defers to domain expertise on domain matters**. The kernel does not override the Security team's authentication design or the Data team's consistency model. The kernel evaluates the STRUCTURAL quality of these designs, not the domain correctness.

4. **Domain teams defer to the kernel on structural matters**. If the Architecture team proposes a design that is structurally incoherent (components do not connect, interfaces do not match), the kernel's structural assessment takes precedence.

5. **Unresolvable conflicts escalate**. If the kernel and a domain team fundamentally disagree and neither can defer, the conflict is escalated to the human operator.

---

## The Commander's Dual Role

The Commander agent serves as both:

1. **The Controller Architect** — running the kernel's controller loop, managing phase transitions, evaluating audit results, and computing reroute targets.
2. **The OS Orchestrator** — dispatching OS teams, managing inter-team dependencies, synthesizing team outputs, and enforcing OS standards.

These are not separate agents. They are the same agent wearing two hats. This is by design:
- The Controller needs to understand domain context to make good reroute decisions.
- The OS Orchestrator needs to understand structural quality to dispatch the right teams.
- Separating them would create a coordination overhead with no benefit.

**Execution model**: The Commander runs the kernel loop as its primary control flow. At each phase, it dispatches OS teams as sub-tasks within the phase. The loop does not advance until all dispatched teams return results.

---

## Canonical Package and Blueprint Protocol: Unified

The kernel's canonical system package and the OS's Blueprint Protocol are the SAME artifact, viewed from two perspectives:

| Kernel View (Structural) | OS View (Domain) | Unified Field |
|---|---|---|
| System identity | Project metadata | `identity` |
| System primitives | Architecture document | `architecture` |
| Candidate set | Design alternatives | `candidates` |
| Comparison matrix | Architecture decision record | `evaluation` |
| Audit vectors | Quality assessment | `audit_results` |
| Evolution ledger | Design history | `evolution` |
| Packaging metadata | Deliverable manifest | `package_metadata` |

The canonical package IS the blueprint. When the kernel finalizes the package, the OS delivers it as the system blueprint. No translation is needed — they were always the same thing.

---

## Integration Verification

After every phase, the bridge verifies integration integrity:

1. **Schema compliance**: Team outputs conform to the expected format for the current phase.
2. **Context consistency**: Team outputs do not contradict the accumulated context from previous phases.
3. **Completeness**: All dispatched teams have returned results. No silent failures.
4. **Merge compatibility**: Multiple team outputs for the same phase can be merged without conflict.

Integration failures are treated as phase validation failures and trigger a retry before advancing.

---

*Kernel-OS Integration Bridge v1.0 — Kernel Integration Layer*
