# QA Standards — SuperArchitect OS

**Version:** 1.0
**Owner:** QA Agent (Team 4)
**Last Updated:** 2026-03-29

---

## Purpose

These standards define the minimum quality bar for every system built by the SuperArchitect OS. They are not aspirational — they are mandatory. A system that fails to meet these standards is not production-ready.

---

## 1. Test Strategy Standards

### 1.1 Test Pyramid

Every system must implement a test pyramid with the following proportions:

| Level | Proportion | Execution Speed | Scope |
|-------|-----------|----------------|-------|
| Unit | 70% | <100ms each | Single function, class, or module |
| Integration | 20% | <5s each | Module + infrastructure (DB, broker, cache) |
| E2E / API | 8% | <30s each | Full system user journeys or API flows |
| Manual / Exploratory | 2% | Varies | Edge cases, UX, non-automatable scenarios |

**Inverting the pyramid is a critical defect.** Systems with more E2E tests than unit tests have slow, brittle, expensive CI pipelines.

### 1.2 Coverage Requirements

| Metric | Minimum | Target |
|--------|---------|--------|
| Line coverage (business logic) | 90% | 95% |
| Line coverage (infrastructure) | 70% | 80% |
| Branch coverage (business logic) | 85% | 90% |
| Critical path coverage | 100% | 100% |
| API endpoint coverage | 100% | 100% |
| Error path coverage | 80% | 90% |

Coverage is measured and enforced in CI. Coverage decreases fail the build.

### 1.3 Test Naming Convention

Tests must be named to describe the behavior being verified:

```
// Pattern: [Unit]_[Scenario]_[ExpectedResult]
test("createOrder_withInvalidItems_throwsValidationError")
test("processPayment_whenGatewayTimesOut_retriesThreeTimes")
test("calculateDiscount_forPremiumUser_appliesTwentyPercent")
```

### 1.4 Test Independence

- Tests must not depend on execution order
- Tests must not share mutable state
- Each test sets up its own fixtures and tears them down
- Parallel test execution must be the default

---

## 2. Unit Testing Standards

### 2.1 What to Unit Test
- All business logic and domain rules
- All data transformations and calculations
- All validation logic
- All state machines and workflow transitions
- All error handling paths
- All edge cases: null, empty, boundary values, overflow

### 2.2 What NOT to Unit Test
- Framework boilerplate (controllers that only delegate)
- Simple getters/setters with no logic
- Third-party library behavior (test your integration, not their code)
- Database queries (test at integration level)

### 2.3 Unit Test Rules
1. **No I/O**: Unit tests must not touch the filesystem, network, or database
2. **No sleep**: Tests that wait for time are flaky by design
3. **Deterministic**: Same input, same output, every time. No randomness without fixed seeds
4. **Fast**: Individual unit test <100ms. Full unit suite <60 seconds
5. **Isolated**: Mock external dependencies. Test the unit in isolation
6. **Assertions are specific**: Assert exact values, not "not null" or "not empty"

### 2.4 Mocking Standards
- Mock at boundaries (ports/interfaces), not internal implementation
- Prefer fakes (in-memory implementations) over mock frameworks when practical
- Verify mock interactions only when the interaction IS the behavior
- Never mock what you do not own — wrap third-party APIs in your own interface

---

## 3. Integration Testing Standards

### 3.1 Scope
Integration tests verify that modules work correctly with real infrastructure:
- Database: real database instance (testcontainers or embedded)
- Message broker: real broker instance
- Cache: real Redis/cache instance
- External APIs: contract-verified stubs or sandbox environments

### 3.2 Rules
1. **Use testcontainers or equivalent**: Spin up real infrastructure in containers
2. **Database tests use transactions**: Roll back after each test for isolation
3. **Test migrations**: Verify that migration scripts produce the expected schema
4. **Test error paths**: Connection failures, timeouts, constraint violations
5. **No shared state**: Each test starts with a clean database state
6. **Execution speed**: <5 seconds per test. Slow integration tests go in a separate suite

### 3.3 Contract Testing
For service-to-service APIs:
- Consumer-driven contract tests are mandatory
- Provider verifies all consumer contracts in CI
- Breaking a consumer contract fails the provider build
- Contract test suite runs in <30 seconds

---

## 4. API Testing Standards

### 4.1 Coverage
Every API endpoint must have tests for:
- Happy path with valid input
- Invalid input (missing fields, wrong types, boundary values)
- Authentication: unauthenticated requests return 401
- Authorization: unauthorized requests return 403
- Rate limiting: excess requests return 429
- Idempotency: repeated requests produce consistent results
- Error responses follow standard format (RFC 7807)

### 4.2 Schema Validation
- Response schemas are validated against OpenAPI specification
- Breaking schema changes fail the build
- Backward compatibility is verified for all versioned APIs

### 4.3 API Test Structure
```
describe("POST /api/v1/orders")
  it("creates order with valid input — returns 201")
  it("rejects empty items array — returns 400")
  it("rejects negative quantity — returns 400")
  it("rejects unauthenticated request — returns 401")
  it("rejects unauthorized user — returns 403")
  it("handles duplicate idempotency key — returns existing order")
  it("returns rate limit error after threshold — returns 429")
```

---

## 5. Performance Testing Standards

### 5.1 Test Types

| Type | Purpose | When to Run |
|------|---------|-------------|
| Load | Verify behavior under expected peak load | Every release |
| Stress | Find the breaking point | Monthly or after major changes |
| Spike | Verify behavior under sudden load increase | Quarterly |
| Soak | Find memory leaks and resource exhaustion | Weekly in staging |
| Capacity | Determine maximum throughput | Before launch, after scaling changes |

### 5.2 Performance Budgets

Define latency budgets per endpoint class:

| Endpoint Class | p50 | p95 | p99 | Throughput |
|---------------|-----|-----|-----|------------|
| User-facing read | <50ms | <200ms | <500ms | Per capacity plan |
| User-facing write | <100ms | <500ms | <1000ms | Per capacity plan |
| Background job | <5s | <15s | <30s | Per SLA |
| Batch processing | <1min | <5min | <15min | Per SLA |

### 5.3 Performance Test Rules
1. Performance tests run against production-like infrastructure (not local dev)
2. Test data volume matches production scale (minimum 10% of production data)
3. Performance regression >10% fails the build
4. Results are stored historically for trend analysis
5. Profiling data is captured for every performance test run
6. Connection pool, thread pool, and memory usage are measured

---

## 6. Security Testing Standards

### 6.1 Automated Security Tests (Every Build)
- OWASP Top 10 vulnerability scanning (DAST)
- Dependency vulnerability scanning (SCA)
- Static application security testing (SAST)
- Secret scanning (no credentials in code or config)
- Container image scanning (no known CVEs above medium)

### 6.2 Security Test Coverage
- [ ] SQL injection on all database-backed endpoints
- [ ] XSS on all user-input-rendering paths
- [ ] CSRF protection on all state-changing endpoints
- [ ] Authentication bypass attempts on all protected endpoints
- [ ] Authorization escalation attempts (horizontal and vertical)
- [ ] Input validation on all external-facing parameters
- [ ] File upload restrictions (type, size, content validation)
- [ ] Rate limiting on authentication endpoints
- [ ] Session fixation and session hijacking prevention

### 6.3 Penetration Testing
- Full penetration test before initial production launch
- Annual penetration test thereafter (or after major architecture changes)
- Findings tracked to resolution with SLA by severity

---

## 7. Chaos Engineering Standards

### 7.1 Steady State Hypothesis
Before injecting failures, define what "normal" looks like:
- Response latency within budgets
- Error rate below threshold (default: 0.1%)
- No data loss or corruption
- Dependent services handle partial failures gracefully

### 7.2 Failure Injection Scenarios

| Scenario | Method | Expected Behavior |
|----------|--------|-------------------|
| Database failure | Kill DB connection | Circuit breaker opens, graceful degradation |
| External service timeout | Inject latency | Timeout triggers, fallback response served |
| Network partition | Block traffic between services | Circuit breaker, retries, eventual recovery |
| Memory pressure | Limit container memory | OOM handled, pod restarts, no data loss |
| Disk full | Fill volume | Writes fail gracefully, alerts fire |
| DNS failure | Block DNS resolution | Cached connections work, new connections fail gracefully |

### 7.3 Chaos Testing Rules
1. Start in staging, graduate to production only with approval
2. Blast radius must be controlled (affect one component, not the whole system)
3. Rollback mechanism must exist before starting
4. Run during business hours when the team is available
5. Document every experiment: hypothesis, method, observations, actions

---

## 8. Test Data Management

### 8.1 Test Data Principles
- Test data is versioned and reproducible
- No production data in test environments (privacy, compliance)
- Synthetic data generators produce realistic data at scale
- Test data includes edge cases: Unicode, long strings, special characters, empty values
- Sensitive data in test environments is anonymized or synthetic

### 8.2 Test Data Strategy
| Environment | Data Source | Volume |
|-------------|-----------|--------|
| Unit tests | In-code fixtures | Minimal |
| Integration tests | Seed scripts + factories | Representative |
| Performance tests | Generated synthetic data | Production-scale |
| Staging | Anonymized production snapshot | Full production volume |

---

## 9. CI/CD Quality Gates

### Gate 1: Pre-Merge (Pull Request)
- [ ] All unit tests pass
- [ ] All integration tests pass
- [ ] Code coverage meets minimum thresholds
- [ ] No new security vulnerabilities (SAST, SCA)
- [ ] No linting errors
- [ ] API contract tests pass

### Gate 2: Post-Merge (Main Branch)
- [ ] Full test suite passes (unit + integration + API + contract)
- [ ] Performance tests show no regression >10%
- [ ] Container image scan clean (no critical/high CVEs)
- [ ] Documentation updated for any API changes

### Gate 3: Pre-Production
- [ ] E2E tests pass in staging
- [ ] Load test passes with production-like data
- [ ] Security scan clean
- [ ] Monitoring and alerting verified
- [ ] Runbook updated for new failure modes
- [ ] Rollback procedure tested

---

## 10. Defect Management

### Severity Classification

| Severity | Definition | Response SLA |
|----------|-----------|-------------|
| P0 - Critical | System down, data loss, security breach | Fix within 1 hour |
| P1 - High | Major feature broken, significant user impact | Fix within 4 hours |
| P2 - Medium | Feature degraded, workaround available | Fix within 1 sprint |
| P3 - Low | Cosmetic, minor inconvenience | Prioritize in backlog |

### Defect Response Protocol
1. Reproduce the defect with a failing test
2. Fix the defect
3. Verify the fix with the test
4. Root cause analysis: why was this not caught earlier?
5. Add the missing test to prevent regression
6. Update test strategy if a class of defects was missed

---

*Quality is not a phase. It is a continuous property maintained at every step of the build process. These standards are the minimum bar — not the ceiling.*
