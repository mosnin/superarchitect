# QA Checklists — SuperArchitect OS

**Version:** 1.0
**Owner:** QA Agent (Team 4)
**Last Updated:** 2026-03-29

---

## Purpose

These checklists are the QA Agent's operational tools for verifying system quality at critical milestones. Every checklist item must be verified — not assumed, not skipped. Mark each item as PASS, FAIL, or N/A with justification.

---

## 1. Pre-Release Checklist

This checklist must be completed before any production deployment.

### 1.1 Code Quality
- [ ] All code passes linting with zero warnings
- [ ] No TODO/FIXME/HACK comments remain in release code
- [ ] Code review completed by at least one engineer who did not write the code
- [ ] No dead code or unused imports
- [ ] All functions have explicit return types (typed languages)
- [ ] Error handling is explicit — no swallowed exceptions
- [ ] Logging follows structured logging standards
- [ ] No hardcoded configuration values — all externalized

### 1.2 Test Coverage
- [ ] Unit test coverage >= 90% on business logic
- [ ] Integration test coverage on all external integration points
- [ ] All API endpoints have happy path and error path tests
- [ ] Contract tests pass for all service-to-service interfaces
- [ ] No flaky tests in the suite (zero tolerance)
- [ ] Test execution time within budget (<5 min for full suite)
- [ ] All new code has corresponding tests
- [ ] Edge cases tested: empty input, boundary values, concurrent access

### 1.3 Security
- [ ] SAST scan clean (no critical/high findings)
- [ ] Dependency scan clean (no known critical CVEs)
- [ ] Container image scan clean
- [ ] Secret scan confirms no credentials in codebase
- [ ] Authentication tested on all protected endpoints
- [ ] Authorization tested for all roles and permissions
- [ ] Input validation on all user-facing parameters
- [ ] HTTPS enforced on all external endpoints
- [ ] CORS configuration reviewed and restricted
- [ ] Rate limiting configured on public endpoints

### 1.4 Performance
- [ ] Load test passes at expected peak traffic
- [ ] No performance regression >10% from previous release
- [ ] Database queries reviewed for N+1 and missing indexes
- [ ] Response times within defined latency budgets
- [ ] Memory usage stable under sustained load (no leaks)
- [ ] Connection pools sized correctly for expected concurrency

### 1.5 Data Integrity
- [ ] Database migrations are backward-compatible
- [ ] Migration rollback script exists and is tested
- [ ] Data validation rules enforced at application and database level
- [ ] Backup and restore procedure verified
- [ ] No data loss paths in error handling

### 1.6 Observability
- [ ] Structured logging produces correct output
- [ ] Distributed tracing propagates across all service boundaries
- [ ] Metrics endpoints expose RED metrics (rate, errors, duration)
- [ ] Health check endpoints respond correctly
- [ ] Alerts configured for all critical failure modes
- [ ] Dashboard shows key metrics for this service
- [ ] Runbook exists for all known failure scenarios

### 1.7 Deployment
- [ ] Deployment is automated (no manual steps)
- [ ] Rollback procedure documented and tested
- [ ] Zero-downtime deployment verified
- [ ] Feature flags configured for gradual rollout (if applicable)
- [ ] Database migrations run before application deployment
- [ ] Post-deployment smoke tests pass
- [ ] Monitoring dashboards accessible during deployment

---

## 2. API Endpoint Checklist

Run this checklist for every new or modified API endpoint.

### 2.1 Design
- [ ] Endpoint follows REST conventions (or documented alternative)
- [ ] URL path uses kebab-case and plural nouns for collections
- [ ] HTTP methods used correctly (GET=read, POST=create, PUT=replace, PATCH=update, DELETE=remove)
- [ ] Query parameters for filtering/sorting/pagination on collection endpoints
- [ ] Response status codes are semantically correct
- [ ] API versioning strategy applied

### 2.2 Input Validation
- [ ] All required fields validated (returns 400 if missing)
- [ ] Field types validated (string, number, date, enum)
- [ ] String lengths bounded (min/max)
- [ ] Numeric ranges bounded (min/max)
- [ ] Email/URL/phone formats validated where applicable
- [ ] Array/list sizes bounded
- [ ] Nested object depth bounded
- [ ] No unexpected fields accepted (strict schema validation)
- [ ] SQL injection prevented (parameterized queries)
- [ ] XSS prevented (output encoding)

### 2.3 Authentication and Authorization
- [ ] Endpoint requires authentication (unless explicitly public)
- [ ] Authentication token validated (signature, expiry, issuer)
- [ ] Authorization checks enforce required permissions
- [ ] Resource-level authorization (user can only access their own data)
- [ ] Admin endpoints are restricted to admin roles
- [ ] Authentication failure returns 401
- [ ] Authorization failure returns 403

### 2.4 Error Handling
- [ ] Validation errors return 400 with specific field-level messages
- [ ] Not found returns 404
- [ ] Conflict returns 409
- [ ] Internal errors return 500 with no internal details leaked
- [ ] Error response format follows RFC 7807 Problem Details
- [ ] All error paths are logged with correlation ID

### 2.5 Response Design
- [ ] Response includes only necessary fields (no over-fetching)
- [ ] Sensitive fields excluded from responses (passwords, internal IDs)
- [ ] Pagination implemented for collection endpoints
- [ ] Consistent date format (ISO 8601)
- [ ] Consistent ID format (UUID or documented alternative)
- [ ] Response documented in OpenAPI specification

### 2.6 Performance
- [ ] Response time within latency budget
- [ ] Database query count is bounded (no N+1)
- [ ] Caching applied where appropriate (Cache-Control headers)
- [ ] Payload size is bounded (large responses paginated)
- [ ] Rate limiting applied

### 2.7 Idempotency
- [ ] POST endpoints support idempotency keys
- [ ] PUT/DELETE endpoints are naturally idempotent
- [ ] Retry-safe: repeated requests produce consistent state

---

## 3. Database Change Checklist

Run this checklist for every database schema change.

### 3.1 Migration Design
- [ ] Migration has a unique, sequential identifier
- [ ] Migration is backward-compatible (old code works with new schema)
- [ ] Migration is forward-compatible (new code works if migration is rolled back)
- [ ] Large table migrations use online DDL or phased approach
- [ ] Migration does not lock tables for extended periods
- [ ] Migration handles existing data correctly (not just new rows)

### 3.2 Rollback
- [ ] Rollback migration script exists
- [ ] Rollback tested in staging
- [ ] Rollback preserves data integrity (no data loss)
- [ ] Rollback procedure documented in runbook

### 3.3 Performance Impact
- [ ] New indexes do not slow critical write paths unacceptably
- [ ] Removed indexes do not slow critical read paths
- [ ] New columns have appropriate defaults (avoid full table rewrite)
- [ ] Query performance tested with production-scale data volume
- [ ] Index usage verified with EXPLAIN ANALYZE

### 3.4 Data Integrity
- [ ] NOT NULL constraints applied where appropriate
- [ ] CHECK constraints validate business rules
- [ ] Foreign key constraints maintain referential integrity
- [ ] Unique constraints prevent duplicate data
- [ ] Default values are semantically correct
- [ ] Enum types or check constraints for status fields

### 3.5 Security
- [ ] PII columns identified and documented
- [ ] Encryption applied to sensitive columns
- [ ] Access controls reviewed for new tables/columns
- [ ] Audit trail requirements satisfied

---

## 4. Feature Checklist

Run this checklist for every new feature before it is considered complete.

### 4.1 Requirements
- [ ] Feature requirements documented and reviewed
- [ ] Acceptance criteria defined and testable
- [ ] Edge cases identified and documented
- [ ] Error scenarios documented with expected behavior
- [ ] Performance requirements defined (latency, throughput)

### 4.2 Implementation
- [ ] Implementation matches accepted architecture design
- [ ] Code follows project coding standards
- [ ] Feature flag wraps the feature (for gradual rollout)
- [ ] Configuration externalized (no hardcoded values)
- [ ] Backward compatibility maintained (existing APIs not broken)

### 4.3 Testing
- [ ] Unit tests cover all business logic paths
- [ ] Integration tests verify infrastructure interactions
- [ ] API tests verify endpoint behavior
- [ ] Error paths tested (what happens when things fail?)
- [ ] Boundary values tested (empty, max, min, null)
- [ ] Concurrent access tested (if applicable)
- [ ] Performance tested under expected load

### 4.4 Security
- [ ] Authentication required on all new endpoints
- [ ] Authorization enforced (correct roles/permissions)
- [ ] Input validated on all new parameters
- [ ] No new dependencies with known vulnerabilities
- [ ] Sensitive data handled according to data classification

### 4.5 Observability
- [ ] Logging added for significant business events
- [ ] Metrics added for feature-specific measurements
- [ ] Tracing covers the full feature execution path
- [ ] Alerts configured for feature-specific failure modes
- [ ] Dashboard updated to show feature metrics

### 4.6 Documentation
- [ ] API documentation updated (OpenAPI spec)
- [ ] README updated if architecture changed
- [ ] Runbook updated for new operational scenarios
- [ ] Architecture Decision Record created for significant choices
- [ ] User-facing documentation updated (if applicable)

---

## 5. Infrastructure Change Checklist

Run this checklist for every infrastructure modification.

### 5.1 Change Management
- [ ] Change defined in Infrastructure as Code (Terraform, Pulumi, CDK)
- [ ] Change reviewed by at least one infrastructure engineer
- [ ] Blast radius assessed (what breaks if this goes wrong?)
- [ ] Rollback plan documented and tested
- [ ] Change window scheduled during low-traffic period

### 5.2 Security
- [ ] IAM permissions follow least privilege
- [ ] Network rules restrict access appropriately
- [ ] Encryption enabled for data at rest and in transit
- [ ] Secrets managed through vault (not environment variables)
- [ ] Security group changes reviewed

### 5.3 Reliability
- [ ] High availability maintained (multi-AZ, replicas)
- [ ] Auto-scaling configured correctly
- [ ] Health checks updated for new infrastructure
- [ ] Backup strategy verified
- [ ] Disaster recovery plan updated

### 5.4 Observability
- [ ] Monitoring covers new infrastructure components
- [ ] Alerts configured for infrastructure-level failures
- [ ] Logs collected from new components
- [ ] Cost monitoring updated

---

## 6. Incident Response Checklist

Run this checklist during and after any production incident.

### 6.1 During Incident
- [ ] Incident commander identified
- [ ] Impact assessed (users affected, data at risk)
- [ ] Communication sent to stakeholders
- [ ] Mitigation actions taken (scale, rollback, failover)
- [ ] Timeline of events recorded
- [ ] External dependencies checked

### 6.2 Post-Incident
- [ ] Root cause identified
- [ ] Blameless post-mortem conducted
- [ ] Action items assigned with owners and deadlines
- [ ] Missing tests or monitors added to prevent recurrence
- [ ] Runbook updated with lessons learned
- [ ] Incident report published to team

---

*Checklists are not bureaucracy. They are the systematic elimination of known failure modes. Pilots, surgeons, and engineers use checklists because the cost of forgetting a step is catastrophic.*
