# Phase 05: Structural Synthesis

> Turn the winning thesis into complete system geometry. Every primitive represented. Every boundary explicit. Every flow traced.

---

## Purpose

Structural Synthesis is where architecture becomes concrete. Phase 4 selected a direction. Phase 5 builds it out into a complete system geometry that accounts for every subsystem, interface, flow, dependency, control point, feedback loop, failure containment mechanism, and evolution path.

This is the most labor-intensive phase in the kernel. It produces the artifact that downstream teams will actually build from. Vagueness here becomes bugs in implementation. Missing interfaces become integration failures. Hidden dependencies become production incidents.

---

## Responsibilities

### 1. Define Subsystems
Every logical grouping of functionality that operates as a unit. Each subsystem must specify:
- **Name and responsibility**: What it does and what it owns
- **Boundary**: What is inside this subsystem and what is outside
- **Inputs**: What it receives and from whom
- **Outputs**: What it produces and for whom
- **State**: What data it owns and how it manages state
- **Failure modes**: How it can break and what happens when it does

### 2. Define Interfaces
Every point where subsystems touch each other. Each interface must specify:
- **Participants**: Which subsystems connect
- **Protocol**: How they communicate (REST, gRPC, message queue, shared database, in-process call)
- **Contract**: What data crosses the boundary, in what format, with what guarantees
- **Failure behavior**: What happens when the interface is unavailable or degraded
- **Versioning strategy**: How the contract evolves over time

### 3. Map Flows
Every significant path that data or control takes through the system. Each flow must specify:
- **Trigger**: What initiates the flow
- **Path**: Ordered list of subsystems and interfaces traversed
- **Transformations**: What happens to data at each step
- **Latency budget**: How much time each step is allocated
- **Failure handling**: What happens if any step fails

### 4. Define Dependencies
Every relationship between subsystems that creates coupling. Dependencies must be:
- **Explicit**: No hidden dependencies through shared state, implicit contracts, or undocumented behavior
- **Directional**: Clear distinction between upstream (depended upon) and downstream (dependent)
- **Classified**: Runtime dependency vs. build-time dependency vs. deployment dependency

### 5. Define Control Points
Places where the system's behavior can be changed without code modification:
- **Configuration gates**: Feature flags, configuration parameters, environment variables
- **Policy injection points**: Where business rules can be updated
- **Rate limiting and throttling**: Where traffic is controlled
- **Circuit breakers**: Where failure propagation is stopped

### 6. Define Feedback Loops
How the system observes itself and responds:
- **Health monitoring**: How each subsystem reports its state
- **Performance feedback**: How the system detects and responds to load changes
- **Quality feedback**: How data quality issues are detected and surfaced
- **Operational feedback**: How operators learn about system behavior

### 7. Define Failure Containment
How failures are prevented from cascading:
- **Blast radius**: For each failure mode, what is affected and what is protected
- **Bulkheads**: Where isolation boundaries prevent failure propagation
- **Graceful degradation**: What reduced functionality is available during partial failure
- **Recovery procedures**: How the system returns to full operation

### 8. Define Evolution Paths
How the system is designed to change over time:
- **Extension points**: Where new capability can be added without modifying existing components
- **Migration paths**: How the system transitions from current state to future state
- **Deprecation strategy**: How old interfaces are retired
- **Scaling strategy**: How the system grows (vertically, horizontally, by decomposition)

---

## System Primitives Checklist

Every synthesis output must account for all 11 universal system primitives. This is not optional.

| Primitive | Represented? | Location in Synthesis |
|-----------|-------------|----------------------|
| Purpose | Required | Top-level objective statement |
| Boundary | Required | System boundary + subsystem boundaries |
| Inputs | Required | System inputs + subsystem inputs |
| Transformations | Required | Flow definitions |
| Outputs | Required | System outputs + subsystem outputs |
| Interfaces | Required | Interface definitions |
| Resources | Required | Resource requirements per subsystem |
| Constraints | Required | Carried from intent + architecture-imposed constraints |
| Feedback | Required | Feedback loop definitions |
| Failure Modes | Required | Failure containment section |
| Evolution Paths | Required | Evolution path section |

If a primitive is genuinely not applicable, it must be explicitly marked `N/A` with justification. Omission without justification is a fail condition.

---

## Kernel Agents

**Primary**: Synthesis Architect — owns the complete structural elaboration, responsible for all 11 primitives being represented.

**Supporting agents activated in parallel**:
- **Failure Mode Architect**: Adversarially pressure-tests every subsystem and interface. For each component, asks "how does this break?" and validates that containment is in place before the phase completes.
- **Optimization Architect**: Reviews the synthesis for unnecessary complexity. Every subsystem, interface, and dependency must earn its place. Removes anything that adds complexity without proportional value.

Domain practitioners translate the synthesized architecture into domain-specific artifacts (engineering specifications, clinical pathway documents, operational manuals, construction plans) — but the synthesis itself is expressed in universal structural primitives.

---

## Output: Synthesized Architecture Object

```yaml
synthesis:
  objective: "<system objective restated>"
  subsystems:
    - name: "<subsystem name>"
      responsibility: "<what it does>"
      boundary: "<inside vs outside>"
      inputs: ["<input 1>"]
      outputs: ["<output 1>"]
      state:
        owned_data: ["<data entity>"]
        storage: "<storage mechanism>"
        consistency: "<strong|eventual|none>"
      failure_modes:
        - mode: "<failure description>"
          blast_radius: "<what is affected>"
          containment: "<how it is contained>"
      resources:
        compute: "<requirement>"
        storage: "<requirement>"
        network: "<requirement>"
  interfaces:
    - id: "<interface id>"
      from: "<subsystem A>"
      to: "<subsystem B>"
      protocol: "<communication protocol>"
      contract:
        request: "<request schema>"
        response: "<response schema>"
        guarantees: "<delivery guarantees>"
      failure_behavior: "<what happens on failure>"
      versioning: "<versioning strategy>"
  flows:
    - name: "<flow name>"
      trigger: "<what initiates>"
      path: ["<subsystem A>", "<interface 1>", "<subsystem B>"]
      latency_budget_ms: 0
      failure_handling: "<strategy>"
  dependencies:
    - from: "<dependent subsystem>"
      to: "<dependency>"
      type: "<runtime|build|deploy>"
      criticality: "<hard|soft>"
  control_points:
    - name: "<control point>"
      type: "<config|policy|rate_limit|circuit_breaker>"
      location: "<where in the system>"
      mechanism: "<how it works>"
  feedback_loops:
    - name: "<feedback loop>"
      signal: "<what is measured>"
      response: "<how the system reacts>"
      latency: "<how fast the loop closes>"
  failure_containment:
    bulkheads: ["<isolation boundary>"]
    circuit_breakers: ["<circuit breaker location>"]
    graceful_degradation:
      - trigger: "<failure condition>"
        reduced_capability: "<what still works>"
    recovery:
      - failure: "<failure type>"
        procedure: "<recovery steps>"
  evolution_paths:
    - name: "<evolution direction>"
      trigger: "<when to evolve>"
      migration: "<how to transition>"
      risk: "<what could go wrong>"
```

---

## Fail Conditions

The phase FAILS if:

1. **Hidden dependencies remain**: Subsystems interact through undocumented channels (shared databases without explicit interface definition, implicit ordering assumptions)
2. **Interface boundaries are unclear**: Contract between subsystems cannot be stated precisely
3. **Feedback logic is weak**: The system cannot observe its own behavior or respond to degradation
4. **Evolution path is absent**: The architecture has no mechanism for change without redesign
5. **System primitives are missing**: Any of the 11 primitives is neither represented nor explicitly marked N/A
6. **Failure containment is absent**: The architecture has no blast radius analysis or graceful degradation strategy

---

## Example: Collaboration Platform Synthesis (Summary)

**Subsystems**: Document Service (CRDT core, document storage, version history), Presence Service (independently scaled, WebSocket management, user state tracking), Collaboration Engine (session management, operation routing, conflict resolution coordination), Event Store (append-only event log, change history, audit trail), API Gateway (authentication, rate limiting, routing), Notification Service (real-time alerts, email digests, webhook delivery)

**Key interfaces**: API Gateway to Document Service (REST), Document Service to CRDT Engine (in-process), Presence Service to Collaboration Engine (gRPC), All services to Event Store (async message queue)

**Key flows**: Collaborative Edit (user keystroke to all participants seeing change, latency budget 200ms), Document Load (user opens document to fully rendered view, latency budget 500ms), Presence Update (user joins/leaves to all participants updated, latency budget 100ms)

**Evolution paths**: Split Document Service by document type (text, spreadsheet, canvas). Add real-time commenting as new subsystem connecting to existing Collaboration Engine. Migrate from single-region to multi-region by promoting Presence Service to regional deployment.

---

## Kernel Agent

**Primary**: Synthesis Architect (`os/kernel/agents/synthesis_architect.md`)

**Supporting**: Failure Mode Architect pressure-tests the synthesis. Optimization Architect reviews for unnecessary complexity.

---

*Phase 05 feeds Phase 06. The synthesis is the architecture. If Phase 06 finds it weak, rerouting returns here for targeted repair.*
