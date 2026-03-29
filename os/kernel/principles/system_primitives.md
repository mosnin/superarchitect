# System Primitives

> The 11 universal structural anchors the kernel uses to reason about any system in any domain.

---

## What Are System Primitives?

System primitives are the irreducible structural elements present in every system. They are not user-facing fields or form inputs. They are cognitive anchors that the kernel uses to ensure no architectural dimension is overlooked.

Every domain differs in content. A hospital management system and a trading platform have different requirements, different technologies, different users. But both have purpose, boundaries, inputs, transformations, outputs, interfaces, resources, constraints, feedback loops, failure modes, and evolution paths. The kernel reasons at this level.

---

## The 11 Primitives

### 1. Purpose
**Why the system exists.** The root objective that justifies the system's existence. Every architectural decision must trace back to purpose. A component that does not serve the purpose is an orphan. A purpose that no component serves is unfulfilled.

### 2. Boundary
**What is inside the system and what is outside.** The line between the system and its environment. Defines what the system owns and what it delegates. Boundaries determine scope, responsibility, and interface surface. Unclear boundaries produce scope creep and integration failures.

### 3. Inputs
**What enters the system.** Data, events, requests, signals, resources that cross the boundary inward. Each input must be characterized: source, format, volume, frequency, reliability. Uncharacterized inputs produce unhandled edge cases.

### 4. Transformations
**What the system does to inputs.** The processing, computation, filtering, enrichment, aggregation, or routing that converts inputs into outputs. Transformations are the system's value creation. They must be explicit -- hidden transformations produce hidden bugs.

### 5. Outputs
**What the system emits.** Data, events, responses, artifacts, side effects that cross the boundary outward. Each output must be characterized: destination, format, latency requirement, durability guarantee. Uncharacterized outputs produce integration failures.

### 6. Interfaces
**Where the system touches other systems, humans, agents, or environments.** The contracts that govern interaction at boundary points. Interfaces define protocol, data format, error handling, versioning, and authentication. Weak interfaces are the primary source of integration failures.

### 7. Resources
**What the system consumes or depends on.** Compute, storage, network, memory, external services, human attention, third-party APIs. Every resource has a capacity, a cost, and a failure mode. Untracked resources produce capacity surprises and cost overruns.

### 8. Constraints
**What limits or shapes the system.** Budget, timeline, team size, regulatory requirements, technology mandates, performance requirements, compatibility requirements. Constraints are not obstacles -- they are design parameters. Ignoring constraints produces unimplementable architecture.

### 9. Feedback
**How the system measures itself and responds.** Monitoring, alerting, auto-scaling, circuit breaking, health checking, performance tracking. Feedback loops close the gap between intended behavior and actual behavior. Systems without feedback are blind.

### 10. Failure Modes
**How the system can break, drift, or degrade.** Every system fails. The question is whether failure is anticipated and contained, or unanticipated and cascading. Each failure mode must specify trigger, blast radius, detection, containment, and recovery.

### 11. Evolution Paths
**How the system should adapt over time.** Extension points, migration strategies, deprecation plans, scaling paths. Systems that cannot evolve become legacy. Evolution paths must be designed, not discovered after the fact.

---

## The Kernel Rule

**No architecture is considered complete unless all 11 primitives are either explicitly represented or intentionally marked not applicable with justification.**

This rule is enforced at Phase 5 (Structural Synthesis) and verified at Phase 6 (Audit). The completeness dimension in the audit vector directly measures primitive coverage.

A primitive marked N/A must include justification. "Evolution paths: N/A because this is a one-time migration tool with defined end-of-life" is acceptable. "Evolution paths: N/A" without justification is not.

---

## How Primitives Are Used

| Phase | Primitive Usage |
|-------|----------------|
| Phase 1 (Intent) | Extract purpose, constraints, inputs/outputs from raw request |
| Phase 2 (Success Model) | Derive quality dimensions from primitive coverage requirements |
| Phase 3 (Search) | Ensure each candidate addresses all primitives differently |
| Phase 4 (Comparison) | Score candidates on primitive completeness |
| Phase 5 (Synthesis) | Build explicit representations of every primitive |
| Phase 6 (Audit) | Verify all primitives are represented with evidence |
| Phase 7 (Packaging) | Ensure package and blueprint cover all primitives |

---

*System primitives are not a checklist. They are a structural language. The kernel speaks this language to reason about any system without domain-specific knowledge.*
