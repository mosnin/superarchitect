# Research Frameworks Reference

Operational reference for the Researcher agent. These frameworks provide structured methodologies for the most common research tasks in the SuperArchitect OS. Apply the appropriate framework based on the research type; combine frameworks when a task spans multiple domains.

---

## 1. Technology Evaluation Framework

### 1.1 Maturity Assessment Criteria

Assess maturity across five dimensions. Each dimension is scored 1–5 using the rubric below.

**Dimension 1: Production Provenance**
- 5 — Used at scale (10M+ users / 10k+ RPS) by multiple independent organizations; case studies available with specifics
- 4 — Used in production by multiple well-known organizations; anecdotal evidence of scale
- 3 — Used in production by smaller organizations; limited scale evidence
- 2 — Early adopters in production; frequent breaking changes; version < 1.0 or unstable API
- 1 — Experimental; no known production use; proof-of-concept stage

**Dimension 2: Community Health**
- 5 — Large, diverse contributor base (100+ contributors in 90 days); responsive maintainers; active ecosystem; issues closed within days
- 4 — Active community (50–100 contributors); reasonable response time; growing ecosystem
- 3 — Moderate community; one or two primary maintainers; issues resolved in weeks
- 2 — Small community; single-maintainer risk; slow issue resolution
- 1 — Abandoned or near-abandoned; issues unaddressed for months; no recent releases

**Dimension 3: Security Track Record**
- 5 — Clean CVE history or CVEs patched within days; regular security audits; explicit security policy; bug bounty program
- 4 — Few CVEs, all medium severity or lower; reasonable patch response time; security policy exists
- 3 — Some CVEs; response time variable; no formal security program
- 2 — Multiple high-severity CVEs; slow patching; no transparent disclosure process
- 1 — Known unpatched critical vulnerabilities; no security disclosure process

**Dimension 4: Performance Benchmarks**
- 5 — Independent benchmarks available with methodology; performs in top quartile for its category; no known performance cliffs
- 4 — Some benchmark data available; good performance characteristics; known performance bounds
- 3 — Limited benchmark data; performance appears adequate from anecdotal evidence
- 2 — Poor benchmark results OR performance data is vendor-only (suspect)
- 1 — Known performance problems for target use case; or no data exists

**Dimension 5: Operational Maturity**
- 5 — First-class observability (metrics, logs, traces); mature upgrade path; active managed offering; excellent documentation
- 4 — Good observability; documented upgrade procedures; reasonable operational tooling
- 3 — Adequate logging; upgrade requires care; some gaps in operational docs
- 2 — Limited observability; upgrades frequently break; sparse operations documentation
- 1 — Black-box operation; no upgrade path; essentially unmanageable at scale

### 1.2 Weighted Scoring Rubric

Default weights (adjust for your context):

| Dimension | Default Weight | High-Security Context | High-Performance Context |
|---|---|---|---|
| Production Provenance | 20% | 15% | 15% |
| Community Health | 15% | 10% | 10% |
| Security Track Record | 20% | 35% | 15% |
| Performance | 20% | 10% | 35% |
| Operational Maturity | 15% | 15% | 15% |
| License & Cost | 10% | 15% | 10% |

**Score interpretation:**
- 4.5–5.0 — Strongly recommended; industry standard for this use case
- 3.5–4.4 — Recommended with noted caveats
- 2.5–3.4 — Conditionally acceptable; specific risks must be mitigated
- 1.5–2.4 — Not recommended; significant concerns require resolution
- < 1.5 — Rejected; do not use

### 1.3 Build vs. Buy vs. Open Source Analysis Framework

Evaluate each option against these criteria:

**Strategic Fit**
- Does ownership of this capability provide competitive differentiation?
- Is this problem core or context (Wardley Maps terminology)?
- What is the cost of vendor dependency for this specific capability?

**Capability Gap Analysis**
- How closely does the available solution match requirements? (0–100% fit)
- What is the cost of the gap: workarounds, missing features, integration complexity?
- Can gaps be closed through configuration vs. code vs. forking?

**Decision Matrix:**

| Factor | Build | Buy | Open Source |
|---|---|---|---|
| Core competitive differentiator | Strong fit | Weak fit | Medium fit |
| Commodity capability | Weak fit | Strong fit | Strong fit |
| Total cost (3-year) | High upfront, low marginal | Predictable, scales with usage | Low upfront, variable maintenance |
| Time to market | Slowest | Fastest | Medium |
| Control & customization | Full | Limited by vendor | Full (with cost) |
| Support burden | Full ownership | Vendor SLA | Community + internal |
| Lock-in risk | None | High | Low |
| Security visibility | Full | Limited | Full |

**Build threshold:** Only build when: (a) the capability is core differentiating AND (b) no OSS/commercial solution achieves ≥80% fit AND (c) team has the expertise.

### 1.4 Total Cost of Ownership (TCO) Framework

3-year TCO calculation template:

```
## Initial Costs
- License / procurement: $X
- Integration / setup engineering: [person-days] × [rate]
- Infrastructure provisioning: $X
- Training and onboarding: [person-days] × [rate]
- Initial data migration (if applicable): [person-days] × [rate]

## Recurring Annual Costs (Year 1 / Year 2 / Year 3)
- License / subscription fees: $X / $X / $X
- Infrastructure (compute, storage, network): $X / $X / $X
- Engineering maintenance (upgrades, patches): [person-days/yr] × [rate]
- Operational support burden: [person-days/yr] × [rate]
- Vendor support contract (if applicable): $X / $X / $X

## Hidden / Risk Costs (probabilistic)
- Probability of major version migration in 3 years: X%
  - If migration required, estimated cost: [person-days] × [rate]
- Vendor price increase risk (SaaS only): X% increase likely
- Security incident response (P × cost): $X

## 3-Year TCO = Initial + Sum(Annual) + Expected(Hidden)
```

---

## 2. Market Research Framework

### 2.1 User Needs Analysis — Jobs-to-be-Done (JTBD)

The JTBD framework focuses on the underlying goal a user is hiring a product to accomplish, not the features they request.

**Job Statement format:**
> When [situation], I want to [motivation], so I can [outcome].

**Research process:**
1. Identify the primary job (functional: what must be done)
2. Identify emotional jobs (how they want to feel during/after)
3. Identify social jobs (how they want to be perceived)
4. Identify the "job executor" (who does the job) vs. "job buyer" (who selects the solution)
5. Map the job steps: define → locate → prepare → execute → confirm → conclude

**Job importance vs. satisfaction matrix:**
- High importance + Low satisfaction = Underserved opportunity (build here)
- High importance + High satisfaction = Table stakes (must match)
- Low importance + Low satisfaction = Ignore
- Low importance + High satisfaction = Over-served (simplify)

### 2.2 Competitive Feature Matrix Methodology

1. **Define evaluation scope**: List the top 3–7 competitors / alternatives
2. **Identify feature dimensions**: From customer requirements, not your roadmap
3. **Score on a consistent scale**: Y (fully supported), P (partial/workaround needed), N (not supported), R (on roadmap)
4. **Document sources and dates** for every cell — competitor features change
5. **Identify pattern clusters**: What do all solutions do? What does nobody do?
6. **Highlight asymmetries**: Where does each solution make a fundamentally different bet?

### 2.3 Technology Trend Analysis

**Gartner Hype Cycle Application:**
- Innovation Trigger: Early research — do not adopt for production
- Peak of Inflated Expectations: Evaluate carefully; beware vendor hype; wait for production evidence
- Trough of Disillusionment: Best time to evaluate seriously — real failure modes are now documented
- Slope of Enlightenment: Good adoption window — best practices emerging, risk understood
- Plateau of Productivity: Safe adoption — well-understood, broadly supported

**ThoughtWorks Technology Radar Rings:**
- Adopt: Industry-tested, suitable for widespread use — strong prior for adoption
- Trial: Ready for use, but not yet broadly adopted — evaluate for your context
- Assess: Interesting, worth understanding — research only, not yet production
- Hold: Proceed with caution; specific concerns identified — require strong justification

**Research protocol for trend analysis:**
1. Locate the technology on current Hype Cycle and Radar if listed
2. Find the earliest production case study (ground truth vs. hype)
3. Search for "lessons learned" and "we migrated away from X" posts
4. Identify the 2–3 organizations whose opinion on this technology you should weight most heavily (domain leaders)

### 2.4 Developer Experience Benchmarking

Measure DX across these dimensions when researching developer tooling:

| Dimension | Metric | How to Research |
|---|---|---|
| Time to Hello World | Minutes from zero to first working output | Follow the official quickstart cold |
| Documentation quality | Completeness, accuracy, searchability | Attempt 3 non-trivial tasks using only docs |
| Error message quality | Actionability of error messages | Deliberately trigger common errors |
| API cognitive load | Number of concepts needed for 80% of tasks | Count core abstractions |
| Community support | Stack Overflow response rate, GitHub issue quality | Sample 10 recent issues |
| Local dev experience | Setup friction, watch/reload, test integration | Full local setup attempt |

---

## 3. Technical Feasibility Framework

### 3.1 Complexity Estimation — T-Shirt Sizing

| Size | Person-Days | Characteristics |
|---|---|---|
| XS | 1–2 | Single, well-understood change; no integration; fully reversible |
| S | 3–10 | Limited scope; 1–2 integration points; team has done similar |
| M | 11–30 | Multiple components; 3–5 integration points; some novel elements |
| L | 31–90 | Cross-team dependencies; significant unknowns; new architectural patterns |
| XL | 91–180 | Major system change; multiple teams; significant integration complexity |
| XXL | 180+ | Organization-scale initiative; fundamental architecture change |

**Complexity drivers (add one size tier for each present):**
- Team has no prior experience with the technology
- Requires migrating existing live data
- Involves external API dependency with unknown reliability
- Requires changes to security boundaries
- No existing test coverage for affected components
- Compliance or regulatory review required

**Anti-patterns in complexity estimation:**
- Anchoring to a similar-sounding but different past task
- Ignoring integration and deployment costs (often 40–60% of total)
- Not accounting for unknown unknowns (add 25% buffer for M+)
- Single-point estimates instead of ranges

### 3.2 Dependency Risk Assessment

For each external dependency, assess:

| Factor | Low Risk | Medium Risk | High Risk |
|---|---|---|---|
| Maintainer bus factor | Large org or foundation | Small team (3+) | Single maintainer |
| Version stability | Stable API, clear versioning | Occasional breaking changes | Frequent breaking changes |
| Our coupling depth | Thin wrapper/abstraction | Direct use in core path | Deep integration, hard to replace |
| Alternatives exist | 3+ viable alternatives | 1–2 alternatives | No viable alternative |
| License risk | Permissive (MIT, Apache) | Weak copyleft (LGPL) | Strong copyleft (GPL) or proprietary |
| Known vulnerabilities | Clean CVE history | Occasional, quickly patched | History of slow patching |

**Aggregate risk score:** Count medium=1, high=2 across all factors.
- 0–2: Low dependency risk
- 3–5: Medium dependency risk — document mitigation
- 6+: High dependency risk — architect for replaceability or find alternative

### 3.3 Integration Complexity Scoring

For each integration point, score:

| Dimension | 1 (Simple) | 2 (Moderate) | 3 (Complex) |
|---|---|---|---|
| Protocol match | Same protocol as our stack | Minor adaptation needed | Protocol translation required |
| Data model alignment | Direct mapping | Minor transformation | Major transformation / impedance mismatch |
| Auth/security | Standard (OAuth, API key) | Custom auth scheme | Deep security integration required |
| Error handling | Standard HTTP/exception patterns | Custom error codes | Complex retry/compensating logic needed |
| Transactionality | Not required | Idempotent operations | Distributed transaction or saga required |
| Operational visibility | Full observability | Partial visibility | Black box |

**Total integration complexity score per point:** Sum of dimensions (6 = simple, 18 = maximum complexity)

### 3.4 Performance Feasibility Analysis

**Process:**
1. State the performance requirement precisely: metric, target value, percentile, conditions
   - Example: "HTTP API p99 response time < 100ms under 1,000 concurrent users"
2. Research known performance characteristics of the proposed stack at the required load
3. Calculate the performance budget: total allowed latency distributed across components
4. Identify the bottleneck hypothesis: which component is most likely to be the constraint?
5. Find benchmark evidence for the bottleneck component specifically
6. Assess gap: estimated performance vs. requirement

**Performance budget template:**
```
Total budget: Xms
- Network (client to edge): Xms
- Load balancer / gateway: Xms
- Application processing: Xms
  - Auth/middleware: Xms
  - Business logic: Xms
  - Database query: Xms
  - External API calls (if any): Xms
- Network (edge to client): Xms
Buffer: 20% of total budget
```

**Feasibility verdict:**
- Estimated ≤ 60% of budget: Feasible with margin
- Estimated 61–90% of budget: Feasible, requires careful implementation
- Estimated 91–120% of budget: Marginal — requires optimization work or requirement revision
- Estimated > 120% of budget: Not feasible without architectural change

---

## 4. Risk Research Framework

### 4.1 Risk Taxonomy for Technical Projects

**Category T — Technical Risks**
- T1: Architecture decision risk (wrong abstraction, premature optimization)
- T2: Technology immaturity risk (framework breaks, API changes)
- T3: Integration risk (external system failures, protocol mismatches)
- T4: Performance risk (doesn't meet SLAs at production load)
- T5: Data integrity risk (corruption, loss, consistency violations)
- T6: Security vulnerability risk (CVEs, design flaws, misconfigurations)

**Category O — Operational Risks**
- O1: Operational complexity risk (too hard to run, debug, upgrade)
- O2: Single point of failure risk (insufficient redundancy)
- O3: Observability gap risk (can't diagnose problems)
- O4: Disaster recovery risk (can't recover from failure)
- O5: Dependency availability risk (external service outage)

**Category K — Knowledge Risks**
- K1: Bus factor risk (knowledge concentrated in one person)
- K2: Skill gap risk (team lacks expertise to implement or operate)
- K3: Documentation debt risk (undocumented system becomes unmaintainable)

**Category E — External Risks**
- E1: Vendor risk (price change, acquisition, sunset, license change)
- E2: Regulatory/compliance risk (new requirement, interpretation change)
- E3: Ecosystem risk (framework abandoned, community fragmentation)

### 4.2 Probability × Impact Matrix

```
         │  Negligible │   Minor    │  Moderate  │   Major    │  Critical  │
         │   Impact 1  │  Impact 2  │  Impact 3  │  Impact 4  │  Impact 5  │
─────────┼─────────────┼────────────┼────────────┼────────────┼────────────┤
Almost   │      5      │     10     │     15     │     20     │     25     │
Certain  │             │            │            │            │            │
─────────┼─────────────┼────────────┼────────────┼────────────┼────────────┤
Likely   │      4      │      8     │     12     │     16     │     20     │
─────────┼─────────────┼────────────┼────────────┼────────────┼────────────┤
Possible │      3      │      6     │      9     │     12     │     15     │
─────────┼─────────────┼────────────┼────────────┼────────────┼────────────┤
Unlikely │      2      │      4     │      6     │      8     │     10     │
─────────┼─────────────┼────────────┼────────────┼────────────┼────────────┤
Rare     │      1      │      2     │      3     │      4     │      5     │
─────────┴─────────────┴────────────┴────────────┴────────────┴────────────┘

Score ≥ 15: CRITICAL — Escalate immediately, must be mitigated before proceeding
Score 10–14: HIGH — Requires active mitigation plan with owner and deadline
Score 5–9: MEDIUM — Document mitigation strategy, monitor
Score 1–4: LOW — Accept or monitor passively
```

### 4.3 Mitigation Options Research

For each HIGH or CRITICAL risk, research:

1. **Avoid** — Can the risk be eliminated by a different design choice? What is the cost?
2. **Transfer** — Can the risk be transferred (insurance, SLA, vendor contract)? At what cost?
3. **Reduce (probability)** — What controls reduce likelihood? What is the residual probability?
4. **Reduce (impact)** — What controls limit the blast radius if the risk materializes?
5. **Accept** — Under what conditions is accepting this risk defensible?

Research source for mitigations: Vendor documentation, security advisories, incident post-mortems (SRE books, public post-mortems), industry standards (NIST, CIS Benchmarks, OWASP).

### 4.4 Known Failure Mode Catalog

Research these failure modes by category when conducting risk assessments:

**Distributed Systems Failures**
- Split-brain in consensus systems (see: Jepsen testing results by Kyle Kingsbury)
- Thundering herd on cache expiry
- Cascading failure through synchronous dependencies
- Deadlock in distributed locking
- Clock skew invalidating TTLs and tokens

**Database Failures**
- Connection pool exhaustion under load spike
- Unindexed queries causing full table scans at scale
- N+1 query patterns discovered only in production
- Lock contention blocking writes
- Backup never tested = no backup

**API / Integration Failures**
- Rate limiting causing silent data loss
- Timeout mismatches between client and server
- Retry storms amplifying downstream failures
- Schema drift between producer and consumer
- Authentication token expiry not handled gracefully

**Security Failures**
- Secrets in environment variables leaked through logs or error messages
- Overly-permissive CORS allowing credential theft
- SQL injection through ORM misuse
- JWT algorithm confusion attacks (alg:none)
- SSRF via user-controlled URL parameters

---

## 5. Knowledge Synthesis Patterns

### 5.1 From Data to Insights to Recommendations

The synthesis hierarchy:

```
Data          → Raw facts, numbers, observations (no interpretation)
Information   → Data with context (what, when, where, how much)
Insights      → Information with meaning (why it matters, what pattern it reveals)
Recommendations → Insights with action (what should be done, by whom, by when)
```

Each level requires more judgment. Document which level your output is operating at. Never present data as a recommendation — show the chain of reasoning.

**Synthesis anti-patterns to avoid:**
- Listing sources without connecting them to a conclusion
- Burying the key finding in section 4 of 6
- Presenting a recommendation without the supporting insight
- Treating a single data point as a pattern
- Confusing correlation for causation in benchmarks or case studies

### 5.2 Structuring Ambiguous Findings

When research produces conflicting or ambiguous results:

1. **State the ambiguity explicitly**: "Sources conflict on X. [Source A] says Y; [Source B] says Z."
2. **Assess source quality**: Apply tier weighting — which source is more authoritative?
3. **Identify the cause of conflict**: Different test conditions? Different versions? Different definitions?
4. **Present both possibilities**: What does each imply for the decision?
5. **Give a provisional recommendation**: "Based on available evidence, lean toward X, but validate assumption Y before committing."

Never resolve ambiguity by picking the more convenient answer. Surface it explicitly.

### 5.3 Confidence Levels in Research Outputs

| Level | Definition | How to Label |
|---|---|---|
| HIGH | Finding supported by primary sources; multiple independent corroborating sources; directly verified | `[CONFIDENCE: HIGH]` |
| MEDIUM | Finding supported by authoritative secondary sources; single primary source; or primary source older than 12 months in a fast-moving domain | `[CONFIDENCE: MEDIUM]` |
| LOW | Finding based on anecdotal or single sources; inference; unverified claim; outdated data | `[CONFIDENCE: LOW]` |
| UNKNOWN | Insufficient data found; unable to assess | `[CONFIDENCE: UNKNOWN — insufficient data]` |

Label every material finding. When the overall brief is MEDIUM or LOW, state this prominently in the header.

### 5.4 Communicating Uncertainty

**Graduated language for uncertainty:**

| Confidence | Language to use | Language to avoid |
|---|---|---|
| HIGH | "X is", "evidence shows", "confirmed by" | N/A — appropriate to state directly |
| MEDIUM | "Evidence suggests", "appears to", "likely" | "X is" (overstates certainty) |
| LOW | "May", "possibly", "one source indicates", "insufficient data to confirm" | "suggests", "appears" (overstates) |
| UNKNOWN | "No data found", "unable to assess" | Speculating without labeling |

**The "So What" test:** For every finding, state what changes in the decision if this finding is wrong. If nothing changes, the finding may not be decision-relevant.

**Escalation triggers:** Escalate to the requesting agent/team when:
- A finding is UNKNOWN for a HIGH-weight decision criterion
- A risk rated HIGH or CRITICAL is discovered mid-research
- The research reveals the framing question is based on a false premise
- Conflicting primary sources cannot be reconciled and the decision hinges on the answer
