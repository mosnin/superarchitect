# Refactoring Workflow

**Version:** 1.0
**Trigger:** `/refactor <scope and description>` or dispatched after audit findings
**Owner:** Commander Agent
**Last Updated:** 2026-03-29

---

## Purpose

This workflow orchestrates systematic refactoring of an existing codebase. Refactoring changes the internal structure of software without changing its external behavior. This workflow ensures refactoring is safe, incremental, and verifiable.

---

## Core Principle

**Refactoring is not rewriting.** The system continues to work at every step. If a refactoring breaks functionality, it was done wrong. Tests are the safety net. No refactoring without tests.

---

## Prerequisites

- Existing test suite with adequate coverage (minimum 80% on areas being refactored)
- Clear definition of what is being refactored and why
- Audit report (if refactoring follows an audit)

---

## Phase 1: Assessment

**Owner:** Architect Agent + Reviewer Agent
**Duration:** Short

**Steps:**
1. Identify the refactoring scope (files, modules, services)
2. Catalog the specific problems being addressed:
   - Code duplication
   - High cyclomatic complexity
   - Poor naming or organization
   - Tight coupling between modules
   - Missing abstractions
   - Performance bottlenecks
   - Security vulnerabilities
   - Outdated patterns or deprecated APIs
3. Verify test coverage in the affected area
4. If coverage is insufficient, write characterization tests first
5. Define success criteria for the refactoring

**Quality Gate:**
- [ ] Refactoring scope clearly defined
- [ ] Problems cataloged with specific file/function locations
- [ ] Test coverage adequate (>= 80% on affected code)
- [ ] Characterization tests written where coverage was insufficient
- [ ] Success criteria defined and measurable

**Artifacts:**
- Refactoring plan document
- Test coverage report for affected areas

---

## Phase 2: Design

**Owner:** Architect Agent
**Duration:** Short

**Steps:**
1. Design the target structure (what the code should look like after)
2. Plan the refactoring sequence (order of changes)
3. Identify risks and mitigation:
   - Database migration risks
   - API contract changes
   - Configuration changes
   - Deployment coordination needs
4. Determine if refactoring can be done incrementally or requires a big-bang change
5. Plan feature flags or branch-by-abstraction for large refactors

**Refactoring Strategies:**
| Strategy | Use When | Risk |
|----------|----------|------|
| Incremental (strangler fig) | Replacing a module or service | Low |
| Branch by abstraction | Changing internal structure behind an interface | Low |
| Parallel implementation | Rewriting a component with verification | Medium |
| Big-bang replacement | No way to incrementally change | High |

**Quality Gate:**
- [ ] Target structure designed and documented
- [ ] Refactoring sequence planned
- [ ] Risks identified with mitigations
- [ ] Strategy selected and justified

---

## Phase 3: Execution

**Owner:** Engineer Agent
**Duration:** Medium to Long

**Rules:**
1. **One refactoring per commit.** Each commit does exactly one thing: rename, extract, move, inline, etc.
2. **Tests pass after every commit.** If tests fail, the commit is wrong — fix before proceeding.
3. **No behavior changes mixed with refactoring.** Behavior changes are separate commits.
4. **Run the full test suite frequently** (at minimum after each logical step).

**Common Refactoring Patterns:**

| Pattern | When to Use |
|---------|-------------|
| Extract Method/Function | Long methods with identifiable sub-operations |
| Extract Class/Module | Class with too many responsibilities |
| Move Method/Function | Method is more related to another class |
| Rename | Name does not communicate intent |
| Replace Conditional with Polymorphism | Complex switch/if-else on type |
| Introduce Parameter Object | Functions with many related parameters |
| Replace Magic Numbers with Constants | Literals scattered through code |
| Extract Interface | Need to decouple from implementation |
| Decompose Conditional | Complex boolean expressions |
| Consolidate Duplicate Code | Same logic in multiple places |

**Steps:**
1. Create refactoring branch
2. Execute refactoring sequence from Phase 2 plan
3. One change per commit with descriptive commit messages
4. Run tests after every change
5. Review intermediate state with Reviewer Agent (for large refactors)
6. Update documentation affected by structural changes

**Quality Gate:**
- [ ] All existing tests pass
- [ ] No behavior changes introduced (verified by tests)
- [ ] Code metrics improved (complexity, duplication, coupling)
- [ ] New structure matches Phase 2 design
- [ ] Documentation updated

---

## Phase 4: Verification

**Owner:** QA Agent + Reviewer Agent
**Duration:** Short

**Steps:**
1. Reviewer Agent performs code review of the entire refactoring
2. QA Agent runs full test suite (unit + integration + E2E)
3. QA Agent compares behavior before and after (contract tests)
4. QA Agent runs performance tests (no regression)
5. Security Agent scans for new vulnerabilities
6. Compare code metrics before and after:
   - Cyclomatic complexity (should decrease)
   - Code duplication (should decrease)
   - Test coverage (should maintain or increase)
   - Module coupling (should decrease)
   - Lines of code (may increase or decrease)

**Quality Gate:**
- [ ] Code review approved
- [ ] All tests pass
- [ ] No behavior changes detected
- [ ] No performance regression
- [ ] Code metrics improved
- [ ] Security scan clean

---

## Phase 5: Deploy and Monitor

**Owner:** DevOps Agent
**Duration:** Short

**Steps:**
1. Merge refactoring branch to main
2. Deploy to staging
3. Run full test suite in staging
4. Deploy to production (using standard deployment strategy)
5. Monitor for 24 hours:
   - Error rates (should be stable)
   - Latency (should be stable or improved)
   - Resource usage (should be stable or improved)
6. If any degradation detected, rollback and investigate

**Quality Gate:**
- [ ] Staging tests pass
- [ ] Production deployment successful
- [ ] No error rate increase in 24-hour monitoring window
- [ ] No latency regression in 24-hour monitoring window

---

## Large-Scale Refactoring: Strangler Fig Pattern

For replacing entire modules or services:

```
Step 1: Identify the boundary of the legacy component
Step 2: Create a new implementation behind the same interface
Step 3: Route a small percentage of traffic to the new implementation
Step 4: Verify correctness (dual-write, shadow mode, or comparison testing)
Step 5: Gradually increase traffic to new implementation
Step 6: Remove legacy implementation when 100% migrated
Step 7: Clean up routing and compatibility code
```

**Duration:** Days to weeks depending on scope.
**Key risk:** Running both old and new implementations increases complexity temporarily.

---

## Anti-Patterns in Refactoring

| Anti-Pattern | Description | Correct Approach |
|-------------|-------------|-----------------|
| Refactoring without tests | Changing code without a safety net | Write characterization tests first |
| Big-bang refactor | Changing everything at once | Incremental changes, one per commit |
| Mixing refactoring with features | Behavior changes + structural changes in one PR | Separate PRs: refactor first, then add features |
| Gold plating | Refactoring code that works and does not need to change | Refactor only what needs to change |
| Scope creep | "While I am here, I will also fix..." | Stick to the plan, log other issues for later |
| No verification | Assuming tests are enough without checking behavior | Compare before/after behavior explicitly |

---

*Refactoring is an investment in the future health of the codebase. It pays dividends in reduced bug rates, faster feature development, and lower cognitive load. But only when done safely and systematically.*
