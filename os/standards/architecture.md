# Architecture Standards — SuperArchitect OS

**Version:** 1.0
**Owner:** SuperArchitect OS Core Team
**Last Updated:** 2026-03-28
**Audience:** All agents and architects operating within the SuperArchitect OS

---

## The 12 Architecture Commandments

These are the immutable principles that govern every architectural decision in the SuperArchitect OS. They are not guidelines — they are laws.

**I. Design for the failure state, not the happy path.**
Every architecture must first answer: "What happens when this component fails?" Systems that only work when everything works are prototypes, not production software.

**II. Boundaries are sacred.**
A boundary drawn is a promise made. Components on opposite sides of a boundary communicate only through the defined interface. What is private stays private. Internal implementation details do not leak across boundaries under any circumstances.

**III. Dependencies flow in one direction.**
High-level policy never depends on low-level details. The direction of dependency is always toward stability. Volatile components depend on stable components, never the reverse. The Dependency Inversion Principle is a constraint, not a suggestion.

**IV. Make the implicit explicit.**
Hidden coupling is more dangerous than visible coupling. State that is shared but undocumented, contracts that are assumed but unwritten, and behaviors that are emergent but unintentional — these are the root of most catastrophic failures. Name everything. Document every contract.

**V. Defer irreversible decisions.**
The longer an irreversible decision can be deferred, the more information will be available when it is made. Build systems that can change their mind about data stores, message brokers, and third-party integrations. Isolate the irreversible parts behind interfaces.

**VI. Complexity is the enemy. Simplicity is the strategy.**
The most dangerous sentence in architecture is "it's not that complicated." Every layer of indirection must earn its place by reducing total system complexity. If an abstraction does not simplify the system, it complicates it.

**VII. Data is the most durable thing you will build.**
Code gets rewritten. Databases outlive applications. Frameworks become obsolete. Data persists. Model your data as if it will outlast everything else, because it will. Schema migrations are permanent decisions.

**VIII. Operational concerns are architectural concerns.**
Deployability, observability, and operability are not afterthoughts. A system that cannot be monitored, deployed, scaled, or debugged in production has failed its most important requirements. Logging, metrics, and tracing are designed in, not bolted on.

**IX. Security is an architectural constraint, not a feature.**
Authentication, authorization, input validation, and secrets management are architectural decisions with architectural consequences. They cannot be added cleanly after the fact. Every boundary is a security boundary until proven otherwise.

**X. Optimize for the engineer who comes next.**
Code is written once and read hundreds of times. Architecture is designed once and operated for years. Every decision should be legible to the engineer who inherits it at 2am on a Sunday. Cleverness is a tax on future maintainers.

**XI. Evolution over revolution.**
Systems that must be completely replaced are systems that were not designed to evolve. Build evolutionary architecture: systems that can absorb change incrementally. Big-bang rewrites almost always fail. Strangler fig patterns almost always succeed.

**XII. Question every assumption.**
The most dangerous architecture decisions are the ones that seem obvious. "Of course it needs a database." "Of course it should be a microservice." Challenge every assumption, document every choice, and revisit every decision as the system evolves.

---

## Coupling and Cohesion

### Definitions

**Cohesion** is the degree to which elements within a module belong together. High cohesion means a module has a single, well-defined responsibility and all its contents serve that responsibility.

**Coupling** is the degree of interdependence between modules. Low coupling means a module can be understood, changed, or replaced with minimal impact on other modules.

The goal is **high cohesion within modules** and **low coupling between modules**.

### Measures

**Cohesion types (lowest to highest):**
1. Coincidental — elements grouped arbitrarily (utility classes with unrelated methods)
2. Logical — elements performing logically similar things grouped together
3. Temporal — elements executed at the same time grouped together
4. Procedural — elements that follow a sequence grouped together
5. Communicational — elements operating on the same data grouped together
6. Sequential — output of one element is input to the next
7. **Functional** — all elements contribute to a single, well-defined function ← Target

**Coupling types (highest/worst to lowest/best):**
1. Content coupling — one module modifies the internals of another (never acceptable)
2. Common coupling — shared global state (avoid)
3. External coupling — shared external format/protocol (acceptable with care)
4. Control coupling — one module controls behavior of another via flags (minimize)
5. Stamp/Data-structured coupling — pass only needed data fields (acceptable)
6. **Data coupling** — pass only simple data parameters (target)
7. **Message coupling** — pure messaging with no shared state (ideal for services)

### Target Ranges

| Metric | Target | Action if Exceeded |
|---|---|---|
| Fan-in (incoming dependencies) | ≤ 10 per module | Investigate if too many callers depend on this — is it a god module? |
| Fan-out (outgoing dependencies) | ≤ 7 per module | Investigate whether module has too many responsibilities |
| Instability ratio (fan-out / (fan-in + fan-out)) | Stable modules: < 0.3; volatile modules: > 0.7 | Rebalance if unstable modules depended upon by many |
| Abstractness | Stable modules should be abstract; concrete modules should be volatile | Refactor if concrete-stable or abstract-volatile |

### Anti-Patterns

- **Inappropriate Intimacy**: Two classes know too much about each other's internals
- **Shotgun Surgery**: One change requires modifications across many unrelated modules
- **Feature Envy**: A method is more interested in data from another class than its own
- **Divergent Change**: One class changes for multiple different reasons
- **Circular Dependencies**: Module A depends on module B which depends on module A

---

## Boundaries

### Identifying System Boundaries

A boundary exists wherever:
1. Two components need to **evolve independently** (different release cadences, different teams)
2. A **technical stack change** occurs (e.g., synchronous to asynchronous, SQL to NoSQL)
3. A **security domain change** occurs (public to internal, user data to admin data)
4. A **scalability requirement change** occurs (different load profiles, different SLAs)
5. An **organizational boundary** exists (Conway's Law — structure follows org structure)

### Boundary Enforcement Rules

- Boundaries are enforced by **interfaces**, not conventions
- No direct object sharing across a boundary — serialize and deserialize
- No shared database tables between bounded contexts (each context owns its data)
- Cross-boundary calls are explicit (function call, HTTP request, message) — never implicit
- Each side of a boundary has its own models; translation happens at the boundary
- Changes to one side of a boundary must not require changes to the other side

### Boundary Granularity Heuristic

Too fine-grained (over-decomposed): If a feature change requires coordinating more than 3 services, the boundary is probably wrong.

Too coarse-grained (under-decomposed): If a single module has more than one team contributing to it, the boundary may need to be drawn more finely.

---

## Dependency Management

### Dependency Direction Rules

```
High-level policy → (interface) ← Low-level details
     Domain         →   Port    ←     Adapter
     Use Case       →   Port    ←   Infrastructure
```

Dependencies always point toward stability and abstraction. Infrastructure components (databases, HTTP handlers, message queues) depend on the domain, never the reverse.

### Dependency Injection

All dependencies are injected, not instantiated. A class should never call `new ConcreteImplementation()` for a dependency it uses but does not own.

```
# Good: Constructor injection
class OrderService:
    def __init__(self, order_repo: OrderRepository, event_bus: EventBus):
        self.order_repo = order_repo
        self.event_bus = event_bus

# Bad: Internal instantiation
class OrderService:
    def __init__(self):
        self.order_repo = PostgresOrderRepository()  # Direct dependency on infrastructure
        self.event_bus = KafkaEventBus()             # Cannot test, cannot swap
```

### Inversion of Control

The framework calls your code; your code does not call the framework. Your domain model must not import framework-specific types. Business logic is framework-agnostic.

### Dependency Versioning Rules

- Pin all external dependencies to exact versions in production
- Use lockfiles (package-lock.json, Pipfile.lock, go.sum, Cargo.lock)
- Automated dependency update PRs (Dependabot or Renovate) required on all repositories
- Major version upgrades require an ADR if they affect public APIs
- Never depend on unreleased or snapshot versions in production

---

## The Architecture Decision Record (ADR) Standard

### When an ADR is Required

An ADR must be filed for any decision that:
- Introduces a new technology, framework, or library to the stack
- Changes how data is stored, queried, or migrated
- Introduces a new architectural pattern or removes an existing one
- Changes how services communicate (sync/async, protocol)
- Has security implications (auth mechanism, encryption approach)
- Cannot be reversed without significant rework
- Resolves a significant disagreement among team members

Rule of thumb: If you'd have to explain the decision to a new team member, write an ADR.

### Full ADR Template

```markdown
# ADR-[NUMBER]: [Short Title]

**Status:** [Proposed | Accepted | Deprecated | Superseded by ADR-NNN]
**Date:** YYYY-MM-DD
**Deciders:** [List of people/agents involved in the decision]
**Consulted:** [List of people/teams consulted]

## Context

[Describe the situation and the forces at play. What problem are we solving?
What constraints exist? What goals are we trying to achieve? Include:
- Business context
- Technical context
- Constraints and non-negotiables
- Options that were considered]

## Decision

[State the decision clearly. What have we decided to do? Be specific enough
that someone reading this in 2 years would understand exactly what was chosen
and could verify whether it was implemented.]

## Rationale

[Explain why this decision was made over the alternatives. Address:
- Why this option over the alternatives
- What tradeoffs are accepted
- What assumptions this decision rests on
- What would cause us to revisit this decision]

## Consequences

### Positive
- [List positive outcomes expected from this decision]

### Negative / Tradeoffs
- [List what we give up or what gets harder]

### Risks
- [List risks and mitigations]

## Compliance

[How do we verify this decision is being followed?
- Architecture fitness function
- Linting rule
- Code review checklist item
- Automated test]

## Related Decisions
- [Links to related ADRs]
```

### ADR Lifecycle

```
Proposed → Accepted → Deprecated
                    ↘ Superseded by ADR-NNN
```

- **Proposed**: Decision is under discussion. Not yet binding.
- **Accepted**: Decision is final. All new work must comply.
- **Deprecated**: Decision no longer applies to new work but existing implementations can remain.
- **Superseded**: A new ADR replaces this one. Link to the superseding ADR.

### Numbering and Storage Convention

- ADRs are numbered sequentially: `ADR-0001`, `ADR-0002`, etc.
- Stored in: `/docs/decisions/ADR-NNNN-short-title.md`
- Index maintained at: `/docs/decisions/README.md`
- ADRs are append-only — never edit an accepted ADR, only supersede it
- ADRs are committed to version control alongside code

---

## Evolutionary Architecture

Evolutionary architecture explicitly supports guided, continuous change across multiple dimensions simultaneously.

### Design Principles for Evolution

1. **Identify fitness functions**: Define what the architecture must preserve as it evolves (see next section)
2. **Isolate irreversible decisions**: Use ports-and-adapters to isolate things that are hard to change
3. **Prefer reversible decisions**: If two options are otherwise equal, choose the more reversible one
4. **Incremental change over big bang**: Strangler fig, branch by abstraction, expand-contract migrations
5. **Data migration strategy**: Every schema change must include a forward and backward migration
6. **Feature flags**: New behaviors introduced behind flags, allowing gradual rollout and rollback

### Patterns for Evolution

- **Strangler Fig**: Build the new system around the old system. Route traffic incrementally. Retire old system component by component.
- **Branch by Abstraction**: Introduce an abstraction, implement new behavior behind it, migrate consumers, remove old implementation.
- **Expand-Contract**: For API changes — expand the contract (add new), migrate clients, contract (remove old).
- **Dark Launching**: Run new code paths in parallel with old, compare results, cut over only when confident.

---

## Architecture Fitness Functions

A fitness function is an objective, automated test that verifies an architectural characteristic is preserved as the system evolves.

### What They Test

Fitness functions test architectural properties, not business logic:
- "No module in the domain layer may import from the infrastructure layer"
- "Average response time for search endpoints must be < 200ms"
- "No cyclic dependencies between modules"
- "All public API endpoints must require authentication"
- "Code coverage must not drop below 85%"

### Implementation

Fitness functions are implemented as:
- **Unit-level**: Static analysis rules (ArchUnit, dependency-cruiser, custom linting rules)
- **Integration-level**: Automated integration tests that verify structural properties
- **Deployment-level**: Pipeline gates that block deployment if structural properties are violated

### Fitness Function Registry

Each architecture document should include a fitness function registry:

```markdown
## Architecture Fitness Functions

| ID | Property | Implementation | Threshold | Blocking |
|----|----------|----------------|-----------|----------|
| FF-01 | No domain → infra dependency | dependency-cruiser rule | Any violation | Yes |
| FF-02 | Response time SLO | k6 load test in CI | p95 < 500ms | Yes |
| FF-03 | Test coverage | Coverage gate in CI | ≥ 85% | Yes |
| FF-04 | No circular deps | Madge / dependency-cruiser | Any cycle | Yes |
```

---

## Technology Radar

The Technology Radar is a curated, opinionated view of which technologies a team or organization should adopt, trial, assess, or hold.

### Four Rings

- **Adopt**: Proven technologies, recommended for widespread use. Default choice.
- **Trial**: Technologies worth pursuing. Use in projects with risk tolerance. Evaluate for Adopt.
- **Assess**: Technologies worth exploring. Understand before committing. Time-box research.
- **Hold**: Technologies to avoid for new projects. Migrate existing usage when practical.

### Four Quadrants

- **Languages & Frameworks**: Programming languages, web frameworks, testing frameworks
- **Tools**: Development tools, CI/CD tools, monitoring tools
- **Platforms**: Cloud platforms, container platforms, data platforms
- **Techniques**: Architectural patterns, development practices, processes

### Maintenance Process

- Radar is reviewed and published quarterly
- Any team member can nominate an item with a written rationale
- Architecture Review Board votes on ring changes
- Hold items require migration roadmap
- Radar is published and accessible to all teams

---

## Architecture Review Board (ARB) Process

### When to Convene

The ARB must review:
- New system design before first production deployment
- Cross-team or cross-domain architectural changes
- Technology additions to the Adopt ring
- Any change affecting > 3 services
- Emergency architectural changes (expedited 24h review)
- Annual review of all systems > 2 years old

### How to Run an ARB Review

**Before the meeting:**
- Submitter files a review request with: context doc, draft ADR(s), fitness function proposal, and risk assessment
- ARB members review materials 48h before the meeting
- ARB chair identifies key questions and circulates them

**During the meeting (90-minute maximum):**
1. Submitter presents the proposal (15 minutes)
2. Clarifying questions from ARB members (15 minutes)
3. Structured discussion of alternatives and concerns (30 minutes)
4. Risk and consequence analysis (15 minutes)
5. Decision: Approve / Approve with conditions / Reject with feedback (15 minutes)

**After the meeting:**
- Decision documented in ADR and signed by ARB chair
- Conditions (if any) tracked as ADR compliance items
- Decision communicated to all affected teams within 24h
- Next review date set if decision is conditional

### ARB Composition

- Architecture lead (chair)
- Security representative
- Platform/infrastructure representative
- One senior engineer from each affected domain
- Optional: product leadership for high-impact decisions

### What the ARB Does NOT Do

- The ARB does not design solutions — it reviews them
- The ARB does not block work — it provides a decision within SLA
- The ARB does not own implementation — submitter owns execution
- The ARB does not micro-manage — it focuses on durable, cross-cutting concerns
