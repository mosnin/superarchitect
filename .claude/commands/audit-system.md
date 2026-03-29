# /audit-system -- System Audit Command

Conduct a comprehensive audit of an existing system.

## Activation Sequence

### Step 1: Boot the OS
1. Read `CLAUDE.md` to activate the SuperArchitect OS
2. Read `os/manifest.md` to load registries
3. Read `os/commander/COMMANDER.md` to instantiate the Commander

### Step 2: Load the Audit Workflow
1. Read `os/workflows/audit.md` to load the 7-phase audit workflow

### Step 3: Execute Phase 1 -- Discovery and Inventory
1. Scan the codebase to identify:
   - Languages, frameworks, and technologies in use
   - Service/module structure
   - Database technologies
   - External dependencies
   - CI/CD configuration
   - Infrastructure definitions
   - Test suites and coverage
   - Documentation
2. Produce system inventory

### Step 4: Execute Phase 2 -- Architecture Audit
1. Read `os/teams/architect/ARCHITECT.md`
2. Read `os/teams/architect/patterns.md`
3. Read `os/standards/architecture.md`
4. Evaluate boundary quality, coupling, cohesion, data architecture, API design, scalability, resilience, and observability
5. Score each criterion and document findings

### Step 5: Execute Phase 3 -- Code Quality Audit
1. Read `os/teams/reviewer/REVIEWER.md`
2. Read `os/standards/quality.md`
3. Evaluate code organization, naming, error handling, test coverage, dependency hygiene, duplication, complexity, and technical debt

### Step 6: Execute Phase 4 -- Security Audit
1. Read `os/teams/security/SECURITY.md`
2. Read `os/teams/security/standards.md`
3. Read `os/teams/security/audit-checklist.md`
4. Evaluate every item on the security audit checklist
5. Check for OWASP Top 10 vulnerabilities
6. Review authentication, authorization, encryption
7. Scan dependencies for known vulnerabilities

### Step 7: Execute Phase 5 -- Performance Audit
1. Read `os/teams/qa/QA.md` and `os/teams/qa/standards.md`
2. Evaluate response times, database query efficiency, caching, resource utilization, and scalability bottlenecks

### Step 8: Execute Phase 6 -- Operational Readiness
1. Read `os/teams/devops/DEVOPS.md` and `os/teams/devops/standards.md`
2. Evaluate CI/CD pipeline, IaC coverage, monitoring, alerting, backup/recovery, incident response

### Step 9: Execute Phase 7 -- Report Synthesis
1. Collect all findings across phases
2. Assign overall system health rating (A through F)
3. Prioritize findings by severity and impact
4. Produce remediation roadmap
5. Present audit report to user

## Usage

```
/audit-system Review the existing codebase in the current directory
```

## Output

The audit produces:
- Overall health rating (A-F)
- Findings by category with severity ratings
- Prioritized remediation recommendations
- Executive summary

$ARGUMENTS
