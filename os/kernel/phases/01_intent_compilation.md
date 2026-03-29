# Phase 01: Intent Compilation

> Convert raw intent into structured architecture input. The messy human request becomes a machine-readable system specification.

---

## Purpose

Intent Compilation is the front door of the kernel. Every architecture project begins here. The phase takes a raw request -- which may be vague, incomplete, contradictory, or overspecified -- and produces a structured intent object that the rest of the pipeline can reason about.

This phase does NOT solve the architecture problem. It defines the problem precisely enough that subsequent phases can solve it. The distinction matters: premature solution thinking at this stage contaminates the search space.

---

## Responsibilities

1. **Identify the core objective**: What does the user actually need? Strip away solution assumptions to find the underlying goal. "I need a highly available cluster" might really mean "I need reliable, scalable deployment infrastructure."

2. **Infer missing structural needs**: Users rarely specify all system requirements. A request for "a real-time collaboration platform" implies persistent transport, conflict resolution, presence management, and storage -- even if none are mentioned.

3. **Identify ambiguity**: Map every point where the request could be interpreted in multiple valid ways. Do not resolve ambiguity by guessing. Flag it explicitly so Phase 2 (Success Model) and Phase 3 (Architecture Search) can account for it.

4. **Capture explicit constraints**: Budget, timeline, team size, regulatory requirements, technology mandates, deployment environment, integration targets. These are hard boundaries the architecture must respect.

5. **Capture implicit constraints**: Organizational maturity, operational capacity, existing infrastructure, team skill profile, maintenance burden tolerance. These shape the architecture even when unstated.

6. **Define requested outputs**: What artifacts does the user expect? A deployment-ready system? An architecture document? A prototype? A migration plan? The output profile shapes every downstream phase.

7. **Estimate consequence level**: How much damage does a wrong architecture decision cause? A personal blog has low consequence. A payment processing system has extreme consequence. This rating affects mode selection and escalation thresholds.

---

## Inputs

| Input | Source | Required |
|-------|--------|----------|
| Raw user request | Human operator | Yes |
| Existing system context | File system / documentation | No |
| Previous system packages | Package archive | No |
| Domain context | Domain practitioner / research | No |
| Requirements framing | Stakeholder input | No |

---

## Outputs

### 1. Intent Object (YAML)

The structured representation of compiled intent. All downstream phases read from this object.

```yaml
intent:
  raw_request: "<original user request verbatim>"
  objective: "<extracted core objective in one sentence>"
  desired_outcomes:
    - "<outcome 1>"
    - "<outcome 2>"
  known_constraints:
    - type: "<budget|timeline|technology|regulatory|team|infrastructure>"
      description: "<constraint description>"
      hard: true|false
  implicit_constraints:
    - type: "<organizational|operational|skill|maintenance>"
      description: "<inferred constraint>"
      confidence: 0.0-1.0
  consequence_level: "<low|medium|high|critical>"
  consequence_rationale: "<why this consequence level>"
  ambiguity_notes:
    - area: "<ambiguous area>"
      interpretations:
        - "<interpretation A>"
        - "<interpretation B>"
      resolution_strategy: "<how to handle>"
output_requirements:
  required_artifacts:
    - "<artifact type>"
  format_preferences:
    - "<format preference>"
  downstream_targets:
    - "<who consumes the output>"
```

### 2. Ambiguity Map

A dedicated structure listing every unresolved ambiguity with its potential impact on architecture decisions. This is separate from the intent object because it is consumed differently -- Phase 3 uses it to generate diverse candidates.

### 3. Requested Output Profile

What the user expects as deliverables. This determines what Phase 7 (Packaging) must produce.

### 4. Consequence Rating

A structured assessment of project consequence that drives mode selection and escalation thresholds throughout the pipeline.

---

## Fail Conditions

The phase FAILS and must not advance if:

1. **Objective remains unclear**: After compilation, the core objective cannot be stated in one clear sentence. The kernel cannot search for architecture without knowing what it is building.

2. **Contradictory goals remain unresolved**: The request contains goals that cannot be simultaneously satisfied (e.g., "zero latency AND complete consistency in a distributed system") and no tradeoff ordering has been established.

3. **Request cannot be represented as a system need**: The request is not a system architecture problem (e.g., "write me a poem"). The kernel exits gracefully.

4. **Consequence level cannot be estimated**: If the kernel cannot determine how much damage a wrong decision causes, it cannot set appropriate thresholds for subsequent phases.

### Fail routing

- If the request is genuinely ambiguous and context is insufficient: escalate to user with specific questions (not open-ended "what do you want?")
- If the request can proceed with explicit uncertainty markers: advance to Phase 2 with ambiguity map attached, allowing the Success Model to account for multiple interpretations

---

## Process

1. Read raw request completely before analysis
2. Extract explicit statements (what the user said)
3. Infer implicit requirements (what the system needs but the user did not say)
4. Identify contradictions and ambiguities
5. Estimate consequence level
6. Gather domain context if needed (from available documentation, domain research, or stakeholder input)
7. Compile intent object
8. Self-review against fail conditions
9. Emit intent object, ambiguity map, output profile, consequence rating

---

## Example

**Raw request**: "Build me a real-time collaboration platform for 10M users"

**Compiled intent object**:

```yaml
intent:
  raw_request: "Build me a real-time collaboration platform for 10M users"
  objective: "Design a system architecture for real-time multi-user collaborative editing at scale"
  desired_outcomes:
    - "Users can simultaneously edit shared documents with sub-second latency"
    - "System handles 10M registered users with concurrent editing sessions"
    - "Changes are persisted durably and consistently"
    - "Platform supports extensibility for future collaboration modes"
  known_constraints:
    - type: "scale"
      description: "Must support 10M users"
      hard: true
  implicit_constraints:
    - type: "operational"
      description: "Requires conflict resolution mechanism for concurrent edits"
      confidence: 0.95
    - type: "infrastructure"
      description: "Requires persistent real-time transport layer"
      confidence: 0.90
    - type: "operational"
      description: "Requires presence/awareness system for active collaborators"
      confidence: 0.85
  consequence_level: "high"
  consequence_rationale: "Data loss or inconsistency in collaborative editing destroys user trust and content integrity"
  ambiguity_notes:
    - area: "collaboration modality"
      interpretations:
        - "Document editing (text-based)"
        - "Visual/canvas collaboration"
        - "General-purpose collaboration framework"
      resolution_strategy: "Generate candidates covering different modalities in Phase 3"
    - area: "10M users concurrency model"
      interpretations:
        - "10M registered, ~100K concurrent"
        - "10M concurrent active sessions"
      resolution_strategy: "Assume 10M registered, estimate concurrent from industry benchmarks"
output_requirements:
  required_artifacts:
    - "system_blueprint"
    - "architecture_decision_records"
    - "handoff_manifest"
  format_preferences:
    - "YAML system package"
    - "Markdown blueprint"
  downstream_targets:
    - "Engineering teams for implementation"
    - "Infrastructure team for provisioning"
```

---

## Kernel Agent

**Primary**: Intent Analyst (`os/kernel/agents/intent_analyst.md`)

The Intent Analyst owns this phase. It applies objective extraction, ambiguity mapping, implied requirement inference, and consequence estimation as its core cognitive functions.

---

*Phase 01 feeds Phase 02. The intent object is the foundation every subsequent phase builds on. A weak intent object produces weak architecture.*
