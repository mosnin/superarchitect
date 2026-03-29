# Phase 03: Architecture Search

> Generate genuinely different structural theses. Not cosmetic variations. Not the same idea with different names. Fundamentally different answers to the same problem.

---

## Purpose

Architecture Search is the creative engine of the kernel. It takes the compiled intent (Phase 1) and success model (Phase 2) and generates 3 or more candidate architectures, each representing a fundamentally different structural thesis about how to solve the problem.

This phase exists because the first architecture idea is rarely the best one. Without structured search, architects anchor on their initial intuition and optimize locally instead of exploring globally. The kernel forces global exploration before local optimization.

---

## Core Requirement: Meaningful Variation

The most critical requirement of this phase is that candidates must be GENUINELY different. Two candidates that both propose microservices with slightly different service boundaries are not meaningfully different. They represent the same structural thesis with cosmetic variation.

Meaningful variation means each candidate makes fundamentally different bets about:
- Where complexity lives (centralized vs. distributed)
- How state is managed (event-sourced vs. stateful vs. stateless)
- Where boundaries are drawn (by domain, by data, by user type, by operation type)
- How components communicate (synchronous vs. asynchronous vs. hybrid)
- What is optimized first (throughput vs. latency vs. consistency vs. simplicity)

### Thesis Axes

Each candidate should be generated from a different structural thesis axis:

| Axis | Thesis | Typical Architecture Shape |
|------|--------|---------------------------|
| **Modularity First** | Decompose into independently deployable units with explicit contracts | Microservices, modular monolith with strict boundaries |
| **Simplicity First** | Minimize moving parts and operational surface area | Monolith, serverless functions, managed services |
| **Robustness First** | Maximize fault tolerance, redundancy, and failure containment | Active-active replication, bulkheaded services, event sourcing |
| **Adaptability First** | Maximize ability to change without redesign | Plugin architecture, event-driven, CQRS with projections |
| **Performance First** | Minimize latency and maximize throughput at the structural level | Edge computing, in-memory processing, co-located services |
| **Cost First** | Minimize operational and development cost | Serverless, managed services, minimal custom code |
| **Data Integrity First** | Maximize correctness and auditability of data flows | Strong consistency, audit logs, immutable event stores |

Not all axes apply to every project. The Search Architect selects 3-5 axes that are most relevant to the project intent and generates one candidate per axis.

---

## Candidate Structure

Each candidate must encode the following elements. This is not optional -- a candidate missing any element is incomplete.

```yaml
candidate:
  candidate_id: "<unique identifier>"
  thesis: "<one-sentence structural thesis>"
  thesis_axis: "<which axis this candidate explores>"
  summary: "<2-3 sentence description of the architecture>"
  assumptions:
    - "<assumption 1: what must be true for this architecture to work>"
    - "<assumption 2>"
  structural_bias: "<what this architecture optimizes for>"
  advantages:
    - "<advantage 1: what this architecture does better than alternatives>"
    - "<advantage 2>"
  liabilities:
    - "<liability 1: what this architecture sacrifices or risks>"
    - "<liability 2>"
  subsystems:
    - name: "<subsystem name>"
      responsibility: "<what it does>"
      boundary: "<what is inside vs. outside>"
  interfaces:
    - from: "<subsystem A>"
      to: "<subsystem B>"
      protocol: "<communication mechanism>"
      contract: "<what is exchanged>"
  flows:
    - name: "<flow name>"
      description: "<what happens>"
      path: ["<subsystem A>", "<subsystem B>", "<subsystem C>"]
  technology_candidates:
    - area: "<area>"
      options: ["<option 1>", "<option 2>"]
      recommendation: "<preferred option and why>"
```

---

## Generation Process

1. Read intent object and success model
2. Identify which thesis axes are most relevant to the project
3. For each selected axis, generate one candidate:
   a. Start from the axis thesis (e.g., "modularity first")
   b. Design a system that maximally expresses that thesis
   c. Identify what the thesis forces you to sacrifice
   d. Encode assumptions, advantages, liabilities
   e. Define subsystems, interfaces, flows at structural level
4. Verify candidates against diversity criteria
5. If candidates are too similar, add another axis or force divergence on a specific structural dimension
6. Dispatch Architecture team for domain-informed refinement
7. Dispatch Research team for technology evaluation
8. Self-review against fail conditions

---

## OS Team Dispatch

### Architecture Team (Team 2)
The Architecture team provides domain-informed candidate generation. While the kernel's Search Architect reasons about structural theses, the Architecture team brings knowledge of proven patterns in the target domain. For a payment system, they know about saga patterns, idempotency keys, and settlement architecture. For a collaboration platform, they know about CRDTs, operational transforms, and presence protocols.

The Architecture team does NOT override the kernel's structural reasoning. It enriches candidates with domain expertise.

### Research Team (Team 10)
Evaluates technology options for each candidate. Provides build-vs-buy analysis. Identifies emerging technologies that might enable novel architectural approaches. Ensures candidates are grounded in available technology rather than theoretical abstractions.

---

## Diversity Validation

Before advancing to Phase 4, the Search Architect must verify that the candidate set has sufficient diversity. The following checks apply:

1. **Thesis distinctness**: No two candidates share the same primary thesis axis
2. **Structural divergence**: Candidates differ in at least 2 of: communication pattern, state management approach, boundary decomposition, deployment topology
3. **Tradeoff coverage**: The candidate set collectively covers the top 3 dimensions in the tradeoff priority ordering -- some candidates optimize for the first, others for the second or third
4. **Assumption variation**: Candidates make different assumptions about constraints, scale, and environment
5. **Liability distribution**: Each candidate's liabilities are different -- they sacrifice different things

If diversity validation fails, the phase enters exploratory iteration: generate additional candidates from unused thesis axes until diversity criteria are met.

---

## Fail Conditions

The phase FAILS if:

1. **Candidates are too similar**: Candidates share the same structural thesis and differ only in naming or minor details
2. **Candidates do not reflect tradeoff differences**: All candidates optimize for the same dimensions, leaving other dimensions unexplored
3. **Candidates are not structurally legible**: A competent architect cannot understand the candidate's system shape from its description
4. **Fewer than 3 candidates generated**: The search space was not adequately explored
5. **Assumptions are unstated**: Candidates lack explicit assumptions, making comparison impossible
6. **Liabilities are hidden**: Candidates present only advantages, concealing structural weaknesses

---

## Example: Real-Time Collaboration Platform

### Candidate 1: Event-Sourced Microservices
- **Thesis**: Decompose by domain responsibility with event sourcing for consistency
- **Axis**: Modularity First
- **Advantages**: Independent scaling, strong audit trail, domain isolation
- **Liabilities**: Operational complexity, eventual consistency challenges, higher infrastructure cost
- **Subsystems**: Document Service, Presence Service, Collaboration Engine, Event Store, API Gateway, Notification Service
- **Key assumption**: Team has microservices operational maturity

### Candidate 2: Modular Monolith with CRDT Core
- **Thesis**: Minimize operational complexity by keeping deployment simple while using CRDTs for conflict-free real-time editing
- **Axis**: Simplicity First
- **Advantages**: Simple deployment, strong consistency via CRDTs, low operational overhead
- **Liabilities**: Scaling ceiling, single deployment unit risk, CRDT complexity in business logic
- **Subsystems**: Monolith Core (Document Module, Presence Module, Auth Module), CRDT Engine, WebSocket Gateway, Storage Layer
- **Key assumption**: Scale can be managed with vertical scaling + read replicas initially

### Candidate 3: Serverless Edge-First Architecture
- **Thesis**: Push collaboration logic to the edge for minimal latency, use serverless for elastic scaling
- **Axis**: Performance First
- **Advantages**: Ultra-low latency, automatic scaling, geographic distribution
- **Liabilities**: Cold start latency, state management complexity at edge, vendor lock-in
- **Subsystems**: Edge Workers (per-region), Central Coordination Service, Durable Object Store, Event Bus, CDN Layer
- **Key assumption**: Cloud provider supports durable objects at edge (e.g., Cloudflare Workers + Durable Objects)

---

## Kernel Agent

**Primary**: Search Architect (`os/kernel/agents/search_architect.md`)

**Supporting**: Failure Mode Architect reviews candidates for obvious structural flaws before advancing to comparison.

---

*Phase 03 feeds Phase 04. The quality of the final architecture is bounded by the quality of the search. A narrow search produces a narrow result.*
