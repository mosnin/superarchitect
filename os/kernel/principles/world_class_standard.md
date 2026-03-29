# World-Class Standard

> World class does not mean maximal complexity. It means fit, quality, coherence, and performance under real constraints.

---

## Definition

A world-class system is one that:
1. Solves the stated problem completely and correctly
2. Does so with structural elegance proportional to the problem's complexity
3. Can be understood by competent engineers
4. Can survive failure without cascading
5. Can evolve without being redesigned
6. Has been pressure-tested against realistic failure scenarios
7. Makes honest claims backed by evidence

World-class is not perfection. It is not maximum complexity. It is not the most advanced technology. It is the right solution, built correctly, for the stated problem, under the stated constraints.

---

## Universal Traits

Every world-class system, regardless of domain, exhibits these traits:

### 1. Coherent
All parts serve the stated objective. No orphaned components. No purpose drift. The system tells a single coherent story from inputs to outputs.

### 2. Adaptable
The system can evolve without fundamental redesign. Change is localized through explicit extension points, versioned interfaces, and migration paths.

### 3. Efficient
Resources are used proportionally to value delivered. No unnecessary complexity. No wasteful duplication. No over-provisioning without justification.

### 4. Failure Aware
The system knows how it can break. Every failure mode is identified with containment, detection, and recovery. Blast radii are bounded. Degradation is graceful.

### 5. Legible
A competent engineer can understand the architecture by reading it. Boundaries are clear. Flows are traceable. Naming is consistent. Decisions are documented.

### 6. Implementable
The architecture can be built with available resources, skills, and timeline. Technology choices are realistic. Team capability requirements are acknowledged.

### 7. Evolvable
The system is designed for change. Extension points exist. Migration paths are defined. Deprecation strategies are planned. The evolution ledger records how the system changed and why.

### 8. Pressure-Tested
The architecture has been adversarially analyzed. Failure modes are identified. Cascade paths are mapped. Overload behavior is characterized. Hidden coupling is detected.

---

## What World-Class Is NOT

### Not maximal complexity
More subsystems, more interfaces, more layers does not make a system world-class. It makes it complex. Complexity must earn its place by solving a real problem.

### Not the latest technology
Using the newest framework, the trendiest architecture pattern, or the most advanced database does not make a system world-class. Technology fitness for the specific problem makes it world-class.

### Not theoretical perfection
A theoretically perfect architecture that cannot be implemented by the available team in the available timeline is not world-class. It is impractical.

### Not a checklist
Having a section for every possible concern (security, observability, resilience, etc.) does not make a system world-class. Structurally addressing those concerns in a way that is coherent with the rest of the architecture does.

### Not overconfident
A world-class architecture acknowledges its uncertainties, documents its assumptions, and identifies where its claims are strongest and weakest.

---

## Anti-Slop Criteria

The kernel rejects outputs that exhibit these defects, regardless of dimensional scores:

1. **Vague**: Architecture descriptions that could apply to any system. "The system uses microservices for scalability" without specifying which services, what boundaries, or what scaling mechanism.

2. **Overabstract**: High-level concepts without structural specifics. "The system employs a robust data layer" without specifying what data, how it flows, where it is stored, or how consistency is maintained.

3. **Checklist-heavy without structure**: Lists of requirements or capabilities without system geometry. "The system supports authentication, authorization, encryption, and audit logging" without specifying where these live in the architecture.

4. **Internally contradictory**: Components that conflict with each other or with stated goals. Strong consistency requirement with eventual consistency architecture. Low-latency requirement with synchronous multi-hop flows.

5. **Underaudited**: Claims without measurable evidence or uncertainty ratings. "The architecture is highly adaptable" without specifying what evidence supports this or what adaptation mechanisms exist.

6. **Overconfident without evidence**: All dimensions scoring above 0.90 with evidence below 0.50. High scores require high evidence.

7. **Too rigid**: Architectures with no evolution paths, no extension points, no migration strategies. Systems that can only change through wholesale redesign.

---

*World-class is a standard, not a label. The kernel measures against this standard at every phase. Architecture that falls short is corrected, not shipped.*
