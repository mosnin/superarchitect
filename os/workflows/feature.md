# Feature Development Workflow

**Version:** 1.0
**Trigger:** `/build-feature <description>` or dispatched by Commander during system build
**Owner:** Commander Agent
**Last Updated:** 2026-03-29

---

## Purpose

This workflow orchestrates the development of a single feature within an existing system. It is lighter than the full system build workflow but maintains the same quality standards.

---

## Prerequisites

- Existing system with established architecture
- Feature description with enough context for requirements
- Commander Agent active

---

## Phase 1: Feature Specification

**Owner:** Product Agent
**Duration:** Short

**Steps:**
1. Product Agent analyzes feature description
2. Product Agent writes user stories with acceptance criteria
3. Product Agent defines success metrics for the feature
4. Product Agent identifies dependencies on existing system components
5. Product Agent prioritizes scope using MoSCoW

**Quality Gate:**
- [ ] User stories have Given/When/Then acceptance criteria
- [ ] Success metrics are measurable
- [ ] Dependencies identified
- [ ] Scope is clear (what is in, what is out)

**Artifacts:**
- Feature specification with user stories
- Success metrics definition

---

## Phase 2: Technical Design

**Owner:** Architect Agent
**Duration:** Short

**Steps:**
1. Architect Agent reviews feature against existing architecture
2. Architect Agent determines:
   - Which services/modules are affected
   - API changes required (new endpoints, modified contracts)
   - Data model changes (new tables, columns, migrations)
   - Event/message changes
   - Infrastructure changes (if any)
3. Architect Agent produces a technical design document
4. Architect Agent identifies breaking changes and migration strategy
5. Security Agent reviews design for security implications

**Quality Gate:**
- [ ] Technical design is compatible with existing architecture
- [ ] Breaking changes identified with migration plan
- [ ] Database migration is backward-compatible
- [ ] API versioning strategy applied if contracts change
- [ ] Security review completed

**Artifacts:**
- Technical design document
- API contract changes (OpenAPI diff)
- Database migration scripts
- ADR (if significant architectural decision made)

---

## Phase 3: Implementation

**Owner:** Engineer Agent
**Duration:** Medium

**Steps:**
1. Engineer creates feature branch
2. Engineer implements database migrations
3. Engineer implements backend changes (domain, API, events)
4. Engineer implements frontend changes (UI, state, integration)
5. Engineer writes unit tests for all new logic
6. Engineer writes integration tests for new integrations
7. Engineer updates API documentation
8. Engineer implements observability (logging, metrics) for new paths

**Quality Gate:**
- [ ] All existing tests still pass (no regressions)
- [ ] New code has unit test coverage >= 90%
- [ ] Integration tests cover new integration points
- [ ] Linting passes
- [ ] Feature flag wraps new behavior (for gradual rollout)

**Artifacts:**
- Implementation code
- Unit and integration tests
- Updated API documentation

---

## Phase 4: Review and QA

**Owner:** QA Agent + Reviewer Agent
**Duration:** Short

**Steps:**
1. Reviewer Agent performs code review
2. QA Agent runs feature checklist from `os/teams/qa/checklists.md`
3. QA Agent verifies acceptance criteria from user stories
4. QA Agent tests edge cases and error paths
5. Security Agent scans for new vulnerabilities
6. QA Agent runs performance test for affected endpoints

**Quality Gate:**
- [ ] Code review approved
- [ ] All acceptance criteria verified
- [ ] Edge cases tested
- [ ] Security scan clean
- [ ] No performance regression > 10%
- [ ] Feature checklist complete

---

## Phase 5: Deploy and Verify

**Owner:** DevOps Agent
**Duration:** Short

**Steps:**
1. Merge feature branch to main
2. CI/CD pipeline runs full test suite
3. Deploy to staging
4. Run smoke tests in staging
5. Deploy to production (canary or feature flag)
6. Verify metrics and monitoring
7. Gradual rollout via feature flag (if applicable)

**Quality Gate:**
- [ ] CI/CD pipeline passes
- [ ] Staging smoke tests pass
- [ ] Production health checks pass
- [ ] No error rate increase
- [ ] Feature metrics tracking correctly

---

## Workflow Summary

```
Phase 1 (Spec)
  │
  ▼
Phase 2 (Design)
  │
  ▼
Phase 3 (Implement)
  │
  ▼
Phase 4 (Review + QA)
  │
  ▼
Phase 5 (Deploy + Verify)
```

**Estimated Duration:** Hours to a day for typical features.

---

*Features are small bets. Each feature follows the same rigor as a full system build, just at a smaller scale. Cut scope, not quality.*
