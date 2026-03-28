# Quality Standards — SuperArchitect OS

**Version:** 1.0
**Owner:** SuperArchitect OS Core Team
**Last Updated:** 2026-03-28
**Audience:** All agents operating within the SuperArchitect OS

---

## Quality Definition

World-class quality in this OS means: **the system does exactly what it must, reliably, securely, and efficiently — and the humans and agents who work with it can understand, change, and trust it.**

Quality is not a phase at the end of development. It is an invariant maintained at every step. A system that "works" but cannot be changed safely is not high quality. A system that is fast but leaks user data is not high quality. A system that is secure but undocumented is not high quality.

**The SuperArchitect OS defines quality across eight inseparable dimensions.** All eight must be satisfied simultaneously. Trading off one dimension against another is a debt that must be consciously incurred and repaid.

---

## Quality Dimensions

### 1. Correctness

The system does what the specification says, handles edge cases gracefully, and never silently produces wrong results.

**Measurement Criteria:**
- Unit test coverage ≥ 90% on all business logic
- Integration test coverage for all external integration points
- Property-based tests for all algorithmic or data-transformation logic
- Zero known unhandled exceptions in production paths
- All edge cases documented and tested: empty inputs, maximum bounds, concurrent access, failure injection
- Contract tests pass for all service interfaces

**Gate:** No feature ships with known correctness gaps. "Probably works" is not acceptable.

---

### 2. Reliability

The system functions continuously under real-world conditions: partial failures, load spikes, bad inputs, and dependency outages.

**Measurement Criteria:**
- Availability target defined and measured (default: 99.9% = 8.7h downtime/year)
- All external calls have timeouts, retries with backoff, and circuit breakers
- Graceful degradation: system degrades partially rather than failing completely
- Chaos engineering tests run quarterly
- MTTR (Mean Time to Restore) ≤ 30 minutes for P1 incidents
- Recovery procedures tested, not just written

**Gate:** Every failure mode must be known and handled.

---

### 3. Performance

The system responds within human-perceptible time limits under expected and peak load.

**Measurement Criteria:**
- Latency targets defined per endpoint class:
  - Interactive user-facing: p50 < 100ms, p95 < 500ms, p99 < 1000ms
  - Background jobs: p95 < 30s
  - Batch: throughput-based, defined per job
- Load tests run against p95 × 2 expected traffic before production launch
- Memory and CPU baselines established and monitored
- Database query performance: no unindexed full-table scans on > 10k row tables
- Performance regression threshold: > 20% degradation triggers rollback

**Gate:** No regression beyond thresholds ships to production.

---

### 4. Security

The system protects data confidentiality and integrity, and resists unauthorized access and manipulation.

**Measurement Criteria:**
- OWASP Top 10 scan clean before every release
- Dependency vulnerability scan: zero critical, zero high-severity unpatched > 14 days
- All secrets in vault or environment — never in source code or logs
- All user input validated and sanitized at boundary
- Authentication and authorization on every API endpoint
- Encryption in transit (TLS 1.2+) and at rest for PII/sensitive data
- Penetration test annually for production systems
- Security review required for ADRs involving auth, data storage, or external integrations

**Gate:** No known security vulnerability ships to production.

---

### 5. Maintainability

The system can be understood, modified, and extended by engineers who did not build it.

**Measurement Criteria:**
- Cyclomatic complexity: function average ≤ 10, no function > 20
- Code duplication: < 5% duplicated blocks (measured by static analysis)
- Function length: median ≤ 20 lines, no function > 60 lines
- Module cohesion: each module has one reason to change
- Naming: all identifiers clearly express intent without abbreviation
- No magic numbers or magic strings in logic
- All TODO/FIXME comments tracked in issue tracker
- Onboarding time target: new engineer productive in ≤ 2 days

**Gate:** Code review must pass maintainability checks.

---

### 6. Usability

The interfaces — user-facing and developer-facing — are intuitive, consistent, and forgiving.

**Measurement Criteria:**
- APIs follow consistent naming and response shape conventions
- Error messages are actionable ("Invalid email format" not "Bad request")
- UIs meet WCAG 2.1 AA accessibility standard
- User-facing flows tested with real users before launch
- Developer experience: SDK/library has working examples for every feature
- Time-to-hello-world for external developers ≤ 15 minutes

**Gate:** Usability review completed before public API launch.

---

### 7. Scalability

The system handles growth in load, data volume, users, and geographic distribution without architectural rework.

**Measurement Criteria:**
- Load profile documented: expected current, 10×, 100× requests/data
- Stateless compute layer (horizontal scale by default)
- Database scaling strategy defined: read replicas, sharding, or caching layer
- Background job system decoupled from request path
- No global locks on hot paths
- Auto-scaling configured and tested
- Capacity planning reviewed every 6 months

**Gate:** Scaling strategy reviewed before launch of any system expected to grow > 10× in 12 months.

---

### 8. Observability

The system's internal state is fully visible to operators and automated systems through logs, metrics, and traces.

**Measurement Criteria:**
- Structured logs (JSON) on all services, with correlation ID on every request
- Metrics exported: request rate, error rate, latency (p50/p95/p99), saturation
- Distributed traces: 100% sampled in dev/staging, ≥ 1% in production (plus error traces)
- Dashboards exist for: system health, user-facing SLOs, dependency health
- Alerting: all SLO breaches page within 5 minutes
- Runbooks linked from every alert
- On-call rotation defined and tested

**Gate:** No system ships to production without dashboards and runbooks.

---

## Quality Gates by Phase

Before advancing from any workflow phase, all gates for that phase must be green.

| Phase | Gate |
|---|---|
| **Requirements** | Acceptance criteria written; non-functional requirements (NFRs) quantified; security requirements identified |
| **Architecture** | ADR filed for major decisions; architecture review passed; fitness functions defined |
| **Implementation** | Unit tests pass; lint clean; no new critical security findings; code review approved |
| **Integration** | Integration tests pass; contract tests pass; performance benchmarks within target |
| **Staging** | Full regression suite green; load test passed; runbooks written; monitoring configured |
| **Production** | Canary deployment passes; SLO dashboards live; on-call briefed; rollback tested |
| **Post-launch** | Postmortem filed for any incidents; tech debt items catalogued; metrics baselined |

---

## Quality Metrics Dashboard

Every system operated under this OS must maintain a live metrics dashboard covering these categories.

### Code Quality Metrics

| Metric | Tool | Target | Alert Threshold |
|---|---|---|---|
| Cyclomatic complexity (avg) | SonarQube / ESLint | ≤ 10 | > 15 |
| Code duplication % | SonarQube / CPD | < 5% | > 10% |
| Test coverage (lines) | Coverage.py / Istanbul | ≥ 85% | < 70% |
| Test coverage (branches) | Coverage tools | ≥ 75% | < 60% |
| Lint violations (errors) | ESLint / Pylint | 0 | > 0 |
| Build pass rate (7d) | CI system | ≥ 95% | < 85% |
| PR cycle time | Git analytics | ≤ 24h | > 72h |
| Open TODO/FIXME count | Custom scan | Trending down | Trending up |

### Runtime Quality Metrics

| Metric | Tool | Target | Alert Threshold |
|---|---|---|---|
| Error rate (5xx) | APM / CloudWatch | < 0.1% | > 0.5% |
| p50 latency | APM | < 100ms | Varies by SLO |
| p95 latency | APM | < 500ms | > SLO limit |
| p99 latency | APM | < 1000ms | > SLO limit |
| Availability (30d rolling) | Synthetic monitoring | ≥ 99.9% | < 99.5% |
| Saturation (CPU/mem) | Infrastructure monitoring | < 70% | > 85% |
| Queue depth | Queue metrics | Stable or shrinking | Growing unboundedly |

### Security Metrics

| Metric | Cadence | Target | Alert Threshold |
|---|---|---|---|
| Critical CVEs unpatched | Continuous | 0 | Any |
| High CVEs unpatched | Continuous | 0 beyond 14 days | > 14 days |
| SAST findings (new) | Per PR | 0 new high/critical | Any new high/critical |
| Dependency age (avg) | Monthly | < 6 months behind latest | > 12 months |
| Secrets scan violations | Per commit | 0 | Any |
| Penetration test age | Annual | < 12 months | > 12 months |

### Operational / DORA Metrics

| Metric | Definition | Elite Target | Alert Threshold |
|---|---|---|---|
| Deployment Frequency | How often code ships to production | Multiple times/day | < once/week |
| Lead Time for Changes | Commit to production time | < 1 hour | > 1 week |
| Change Failure Rate | % deployments causing incident | < 5% | > 15% |
| MTTR | Time from incident start to resolution | < 1 hour | > 24 hours |
| Mean Time Between Failures | Average uptime between incidents | > 30 days | < 7 days |

---

## Technical Debt Framework

### Definition

Technical debt is the gap between the current implementation and the ideal implementation, measured in future development cost and risk. It has two types:

- **Deliberate debt**: Consciously accepted shortcuts with a repayment plan
- **Accidental debt**: Discovered after the fact through code review, incidents, or analysis

### Measurement

Debt items are classified and sized:

| Class | Description | Cost Unit |
|---|---|---|
| **Architectural debt** | Wrong structural decisions, wrong technology | Weeks to refactor |
| **Design debt** | Poor abstractions, wrong patterns | Days to refactor |
| **Code debt** | Low quality code, missing tests | Hours to fix |
| **Documentation debt** | Missing or wrong docs | Hours to write |
| **Dependency debt** | Outdated libraries | Hours to upgrade |

Each debt item is estimated in developer-days to resolve and given a risk score (1–5) for probability and impact of leaving it unresolved.

**Debt Priority = Risk Score × Estimated Growth Rate**

### Prioritization Matrix

| Quadrant | Action |
|---|---|
| High impact, low cost | Fix immediately |
| High impact, high cost | Schedule for next major investment cycle |
| Low impact, low cost | Fix during slack time (20% rule) |
| Low impact, high cost | Monitor and accept |

### Allocation

- **15% of all sprint capacity** is allocated to technical debt reduction
- **100% of hotfix time** is paired with debt item creation if the hotfix introduced shortcuts
- Debt backlog reviewed monthly; any item over 90 days old gets escalated

---

## Code Review Quality Standards

A great code review is not a style check. It is a collaborative act of quality assurance and knowledge transfer.

**What a reviewer must verify:**

1. **Correctness**: Does the code do what the ticket/spec says? Are edge cases handled?
2. **Tests**: Are tests present, meaningful, and covering the right scenarios? Not just "does coverage increase" but "does coverage test the right things"?
3. **Design**: Is this the right abstraction? Will this be maintainable in 6 months?
4. **Security**: Any injection risks, auth bypass possibilities, data exposure vectors?
5. **Performance**: Any obvious N+1 queries, blocking I/O, unindexed queries, memory leaks?
6. **Observability**: Are new paths logged? Are errors surfaced to monitoring?
7. **Documentation**: Is the code self-documenting? Are non-obvious decisions explained in comments?

**Reviewer conduct rules:**
- Comment on code, never on the author
- Distinguish blockers from suggestions: prefix suggestions with "nit:" or "optional:"
- Explain the why, not just the what: "This could cause N+1 queries when..." not "Wrong"
- Approve only when all blockers are resolved
- Target 24-hour turnaround on review requests
- First-time reviewers on a codebase should review with a senior mentor

**Author conduct rules:**
- PR size: ≤ 400 lines changed (excluding generated code and lock files)
- Every PR has a description explaining why, not just what
- Respond to all review comments, even to say "acknowledged, not changing because..."
- Do not merge without at least one approval from a domain-knowledgeable reviewer

---

## Documentation Quality Standards

Documentation is a product. It has users, requirements, and quality criteria.

**The four properties of world-class documentation:**

1. **Findable**: Users can locate the right document within 60 seconds
2. **Accurate**: Every statement is factually correct as of the last-updated date
3. **Complete**: Covers all cases the target audience needs to handle their job
4. **Current**: Updated within 30 days of any change it describes

**Writing standards:**
- Use active voice: "The service authenticates requests" not "Requests are authenticated by the service"
- Lead with the most important information (inverted pyramid)
- One concept per paragraph
- Code examples for every API, configuration, or command
- Avoid jargon; define unavoidable jargon inline on first use
- Use numbered lists for sequential steps, bullet lists for unordered items

**Required metadata on every document:**
```
Title, Purpose, Audience, Last Updated, Owner, Review Frequency
```

---

## The Quality Manifesto

The SuperArchitect OS makes ten commitments to quality on behalf of every system it helps build:

1. **We define quality before we write code.** Acceptance criteria and non-functional requirements are written before implementation begins.

2. **We automate quality checks.** If a quality check can be automated, it must be. Manual checks are fallbacks, not primary mechanisms.

3. **We treat tests as first-class code.** Test code is reviewed, refactored, and maintained with the same rigor as production code.

4. **We do not ship known defects.** Known bugs are either fixed or explicitly deferred with a tracked issue and a risk acceptance sign-off.

5. **We measure what we care about.** Every quality claim is backed by a metric. "The system is fast" is not acceptable without p50/p95/p99 data.

6. **We make security non-negotiable.** There is no business priority that overrides a critical security vulnerability in production.

7. **We pay technical debt.** Fifteen percent of every sprint is reserved for debt reduction. Debt is a liability, not a feature.

8. **We own quality end to end.** Quality is not the QA team's job. Every agent, every developer, every architect owns quality in their domain.

9. **We learn from failure.** Every production incident results in a blameless postmortem with action items that prevent recurrence.

10. **We leave codebases better than we found them.** Every interaction with a codebase either improves its quality or leaves it unchanged. Never worse.
