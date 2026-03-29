# Research Sprint Workflow

**Version:** 1.0
**Trigger:** `/research <topic or question>`
**Owner:** Commander Agent
**Last Updated:** 2026-03-29

---

## Purpose

This workflow orchestrates a focused research sprint to produce decision-quality intelligence on a specific topic. Research sprints answer questions like: "What technology should we use for X?", "How do comparable systems solve Y?", "What are the trade-offs of approach A vs. B?"

---

## Prerequisites

- Clear research question or topic from human operator
- Context about why this research is needed (decision it supports)
- Scope constraints (time budget, depth expectations)

---

## Phase 1: Frame the Research Question

**Owner:** Commander + Researcher Agent
**Duration:** Short

**Steps:**
1. Commander parses the research request
2. Researcher Agent refines the question:
   - What specific decision does this research support?
   - What are the evaluation criteria?
   - What constraints exist (budget, timeline, team skills, compliance)?
   - What is the current state of knowledge?
   - What would change our mind (what evidence would matter)?
3. Researcher Agent produces a research brief

**Quality Gate:**
- [ ] Research question is specific and answerable
- [ ] Evaluation criteria defined
- [ ] Constraints documented
- [ ] Scope appropriate for the time budget

**Artifacts:**
- Research brief (question, criteria, constraints, scope)

---

## Phase 2: Landscape Survey

**Owner:** Researcher Agent
**Duration:** Medium

**Steps:**
1. Identify candidate solutions, technologies, or approaches
2. For each candidate, gather:
   - Core capabilities and architecture
   - Maturity level (experimental, production-ready, legacy)
   - Community and ecosystem health
   - Licensing and cost model
   - Adoption trends (growing, stable, declining)
   - Notable users and case studies
3. Produce a landscape map of all candidates

**Evaluation Dimensions:**
| Dimension | Questions |
|-----------|----------|
| Functionality | Does it solve the problem? What gaps exist? |
| Performance | Can it handle our scale requirements? |
| Reliability | What is its track record in production? |
| Security | What is its security posture? Known vulnerabilities? |
| Operability | How hard is it to deploy, monitor, and maintain? |
| Community | Active development? Responsive maintainers? |
| Cost | License fees? Infrastructure costs? Engineering effort? |
| Team Fit | Does the team have experience? Learning curve? |
| Ecosystem | Integrations, plugins, tooling availability? |
| Longevity | Will this be maintained in 3-5 years? |

**Artifacts:**
- Landscape survey document
- Candidate comparison matrix

---

## Phase 3: Deep Evaluation

**Owner:** Researcher Agent + Architect Agent
**Duration:** Medium

**Steps:**
1. Select top 2-3 candidates from landscape survey
2. For each finalist:
   - Review documentation and architecture in depth
   - Analyze source code quality (if open source)
   - Evaluate integration with our existing stack
   - Identify operational requirements (monitoring, scaling, backups)
   - Assess migration path (from current state to adoption)
   - Estimate total cost of ownership (TCO) over 3 years
3. Architect Agent evaluates architectural fit
4. Security Agent evaluates security implications
5. Build proof-of-concept if needed (time-boxed to hours, not days)

**Proof of Concept Criteria:**
- Focus on the highest-risk assumption
- Time-boxed (maximum 4 hours)
- Produces measurable results (benchmark, feasibility confirmation)
- Throwaway code (not a prototype to evolve into production)

**Artifacts:**
- Deep evaluation report per finalist
- TCO analysis
- Proof-of-concept results (if applicable)
- Architectural fit assessment

---

## Phase 4: Recommendation

**Owner:** Researcher Agent + Commander
**Duration:** Short

**Steps:**
1. Researcher Agent synthesizes findings into a recommendation
2. Recommendation includes:
   - Recommended option with clear rationale
   - Runner-up option as a fallback
   - Risks and mitigations for the recommendation
   - Migration/adoption plan (high-level)
   - Decision criteria that would change the recommendation
3. Commander reviews for completeness and balance
4. Produce Architecture Decision Record (ADR)

**Recommendation Format:**
```markdown
## Recommendation: [Option Name]

### Context
[Why this research was needed]

### Decision
[What we recommend and why]

### Evaluation Summary
| Criterion | Option A | Option B | Option C |
|-----------|---------|---------|---------|
| ...       | ...     | ...     | ...     |

### Trade-offs
[What we gain and what we give up]

### Risks
[What could go wrong and how we mitigate]

### Adoption Plan
[High-level steps to adopt the recommendation]

### Reversibility
[How hard is it to change course if this does not work out]
```

**Quality Gate:**
- [ ] Recommendation is based on evidence, not opinion
- [ ] Multiple options evaluated (minimum 2)
- [ ] Trade-offs explicitly documented
- [ ] Risks identified with mitigations
- [ ] ADR produced for the decision record

**Artifacts:**
- Research recommendation document
- Architecture Decision Record (ADR)
- Supporting evidence and references

---

## Phase 5: Knowledge Base Update

**Owner:** Researcher Agent
**Duration:** Short

**Steps:**
1. Update `os/knowledge/tech-radar/` with evaluations
2. Add case studies to `os/knowledge/case-studies/` if applicable
3. Add anti-patterns discovered to `os/knowledge/anti-patterns.md`
4. Update relevant team files with new patterns or standards
5. File the ADR in `os/knowledge/decision-records/`

**Artifacts:**
- Updated knowledge base files

---

## Research Types

### Technology Evaluation
Focus on: capabilities, performance, ecosystem, cost, team fit.
Example: "Should we use PostgreSQL or CockroachDB for our multi-region database?"

### Architecture Decision
Focus on: patterns, trade-offs, scalability, maintainability.
Example: "Should we use event sourcing or traditional CRUD for the order service?"

### Build vs. Buy Analysis
Focus on: TCO, customization needs, vendor risk, time-to-market.
Example: "Should we build our own auth system or use Auth0?"

### Competitive Analysis
Focus on: market landscape, feature comparison, differentiation opportunities.
Example: "How do competitors handle real-time collaboration?"

### Incident Post-Mortem Research
Focus on: root cause patterns, prevention strategies, industry best practices.
Example: "How do other organizations prevent cascading failures?"

---

## Anti-Patterns in Research

| Anti-Pattern | Description | Correct Approach |
|-------------|-------------|-----------------|
| Analysis paralysis | Researching forever, never deciding | Time-box research, decide with available data |
| Confirmation bias | Seeking evidence for a pre-chosen option | Define evaluation criteria before evaluating |
| Hype-driven | Choosing technology because it is trendy | Evaluate against your specific constraints |
| Resume-driven | Choosing technology to learn something new | Evaluate based on team and business needs |
| Vendor capture | Choosing based on sales pitch | Evaluate with hands-on testing and reference checks |
| Ignoring TCO | Comparing only license costs | Include: engineering time, ops cost, learning curve, risk |

---

*Good research produces clarity. Great research produces confidence in decisions under uncertainty. The goal is not to find the perfect answer — it is to find the best answer given the constraints.*
