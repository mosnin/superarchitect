# System Audit Workflow

**Version:** 1.0
**Trigger:** `/audit-system <path or description>`
**Owner:** Commander Agent
**Last Updated:** 2026-03-29

---

## Purpose

This workflow conducts a comprehensive audit of an existing system. It evaluates architecture, code quality, security, performance, observability, and operational readiness. The output is a detailed report with findings, severity ratings, and prioritized remediation recommendations.

---

## Prerequisites

- Access to the system's source code repository
- Access to documentation (if any)
- System description or context from human operator

---

## Phase 1: Discovery and Inventory

**Owner:** Commander + Architect Agent
**Duration:** Short

**Steps:**
1. Commander scans the codebase to build a system inventory:
   - Languages and frameworks in use
   - Service/module boundaries
   - Database technologies
   - External dependencies
   - CI/CD configuration
   - Infrastructure definitions
   - Test suites
   - Documentation
2. Commander identifies the system type and applicable standards
3. Commander loads appropriate audit checklists

**Artifacts:**
- System inventory document
- Technology stack summary
- Dependency list

---

## Phase 2: Architecture Audit

**Owner:** Architect Agent
**Duration:** Medium

**Evaluation Criteria:**
1. **Boundary Quality**: Are service/module boundaries well-defined? Do they align with domain concepts?
2. **Coupling Analysis**: Are components loosely coupled? Can they change independently?
3. **Cohesion Analysis**: Does each module have a single, clear responsibility?
4. **Data Architecture**: Is the data model appropriate? Are schemas well-designed?
5. **API Design**: Do APIs follow REST/gRPC best practices? Are they consistent?
6. **Scalability**: Can the system handle 10x current load? What are the bottlenecks?
7. **Resilience**: How does the system handle partial failures?
8. **Observability**: Are logs, metrics, and traces present and useful?

**Scoring:**
| Rating | Criteria |
|--------|----------|
| Excellent | Meets all best practices, no significant issues |
| Good | Minor issues, overall sound design |
| Adequate | Functional but has notable gaps |
| Poor | Significant issues requiring remediation |
| Critical | Fundamental problems requiring redesign |

**Artifacts:**
- Architecture audit report with per-criterion scoring
- Architecture diagrams (as-is)
- Recommended improvements prioritized by impact

---

## Phase 3: Code Quality Audit

**Owner:** Reviewer Agent + Engineer Agent
**Duration:** Medium

**Evaluation Criteria:**
1. **Code Organization**: Consistent structure, clear module boundaries
2. **Naming Conventions**: Clear, consistent, self-documenting names
3. **Error Handling**: Explicit, consistent, no swallowed exceptions
4. **Test Coverage**: Unit, integration, and E2E test adequacy
5. **Test Quality**: Tests verify behavior, not implementation; no flaky tests
6. **Dependency Hygiene**: Up-to-date dependencies, no known vulnerabilities
7. **Code Duplication**: DRY compliance, shared utilities where appropriate
8. **Complexity**: Cyclomatic complexity, deep nesting, long functions
9. **Documentation**: Inline docs, README, API docs, architecture docs
10. **Technical Debt**: TODOs, FIXMEs, deprecated patterns, workarounds

**Automated Checks:**
- Linter analysis (eslint, pylint, golangci-lint, or equivalent)
- Code complexity metrics (cyclomatic complexity, cognitive complexity)
- Dependency vulnerability scan (Snyk, Trivy)
- Test coverage report
- Dead code analysis
- Duplication detection

**Artifacts:**
- Code quality report with metrics and findings
- Technical debt inventory
- Prioritized remediation recommendations

---

## Phase 4: Security Audit

**Owner:** Security Agent
**Duration:** Medium

**Process:**
1. Security Agent loads `os/teams/security/audit-checklist.md`
2. Security Agent evaluates every checklist item
3. Security Agent runs automated scans:
   - SAST (static analysis)
   - SCA (dependency vulnerabilities)
   - Container image scanning
   - Secret scanning
   - IaC policy scanning
4. Security Agent reviews:
   - Authentication implementation
   - Authorization implementation
   - Input validation
   - Data encryption
   - Secrets management
   - Network security
5. Security Agent performs threat model review

**Artifacts:**
- Security audit report with findings by severity
- Vulnerability list with CVSS scores
- Threat model assessment
- Remediation plan with SLAs

---

## Phase 5: Performance Audit

**Owner:** QA Agent
**Duration:** Medium

**Evaluation Criteria:**
1. **Response Times**: Are endpoints within latency budgets?
2. **Database Performance**: N+1 queries, missing indexes, slow queries
3. **Caching**: Is caching applied where beneficial? Is invalidation correct?
4. **Resource Utilization**: CPU, memory, connection pool efficiency
5. **Scalability Bottlenecks**: What fails first under load?
6. **Frontend Performance**: Core Web Vitals, bundle size, rendering

**Automated Checks:**
- Load test against staging environment
- Database query analysis (slow query log, EXPLAIN)
- Frontend Lighthouse audit
- Memory leak detection under sustained load

**Artifacts:**
- Performance audit report
- Bottleneck analysis
- Optimization recommendations prioritized by impact

---

## Phase 6: Operational Readiness Audit

**Owner:** DevOps Agent
**Duration:** Short

**Evaluation Criteria:**
1. **CI/CD Pipeline**: Automated? Fast? Reliable? Quality gates?
2. **Infrastructure as Code**: All infra defined in code? Drift free?
3. **Monitoring**: Metrics, logs, traces present? Dashboards useful?
4. **Alerting**: Alerts actionable? Runbooks linked? On-call defined?
5. **Backup/Recovery**: Backups tested? RTO/RPO defined and achievable?
6. **Incident Response**: Process defined? Roles assigned?
7. **Documentation**: Runbook, architecture docs, deployment guide

**Artifacts:**
- Operational readiness report
- Missing operational capabilities list
- Runbook gap analysis

---

## Phase 7: Report Synthesis

**Owner:** Commander
**Duration:** Short

**Steps:**
1. Commander collects all phase reports
2. Commander synthesizes findings into a unified audit report
3. Commander assigns overall system health rating
4. Commander prioritizes all findings by severity and impact
5. Commander produces remediation roadmap

**Overall Health Rating:**
| Rating | Criteria |
|--------|----------|
| A (Excellent) | Production-ready, minor improvements only |
| B (Good) | Production-acceptable, some improvements recommended |
| C (Adequate) | Functional but risks present, improvements needed |
| D (Poor) | Significant risks, remediation required before production |
| F (Critical) | Not production-ready, fundamental issues must be addressed |

**Final Artifacts:**
- `docs/audit/report.md` — Complete audit report
- `docs/audit/findings.md` — All findings with severity and remediation
- `docs/audit/roadmap.md` — Prioritized remediation roadmap
- Executive summary (one-page)

---

## Workflow Diagram

```
Phase 1 (Discovery)
  │
  ├──────────────────────────────────────────┐
  │                                          │
  ▼                                          ▼
Phase 2 (Architecture) ──── Phase 3 (Code Quality)
  │                          │
  ▼                          ▼
Phase 4 (Security) ──────── Phase 5 (Performance)
  │                          │
  ▼                          ▼
Phase 6 (Ops Readiness) ────┘
  │
  ▼
Phase 7 (Report Synthesis)
```

Phases 2-3 run in parallel. Phases 4-6 run in parallel. Phase 7 waits for all.

---

*An audit is not criticism. It is a health check. Every system benefits from periodic, systematic evaluation by fresh eyes applying rigorous standards.*
