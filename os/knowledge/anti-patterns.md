# Anti-Pattern Catalog — SuperArchitect OS

**Version:** 1.0
**Owner:** Research Agent (Team 10)
**Last Updated:** 2026-03-29

---

## Purpose

This catalog documents common anti-patterns encountered in software systems. Each entry describes the pattern, explains why it is harmful, how to detect it, and what to do instead. Anti-patterns are not hypothetical — they are recurring mistakes observed in real systems.

---

## 1. Architecture Anti-Patterns

### 1.1 Distributed Monolith

**Description:** A system deployed as microservices but tightly coupled so that all services must be deployed together, share a database, or require synchronous chains to function.

**Symptoms:**
- Deploying one service requires deploying others
- Services share a database (direct table access across services)
- A single request traverses 5+ services synchronously
- A failure in one service cascades to all others
- Teams cannot make changes without coordinating with other teams

**Root Cause:** Service boundaries drawn around technical layers (API, logic, data) instead of business domains. Or services extracted from a monolith without decoupling the data.

**Remedy:**
- Redraw boundaries around bounded contexts
- Database per service (migrate data ownership)
- Replace synchronous chains with async events where possible
- Apply circuit breakers and bulkheads

### 1.2 Big Ball of Mud

**Description:** A system with no discernible architecture. Everything depends on everything. No module boundaries. Changes in one place break things in unrelated places.

**Symptoms:**
- Cannot describe the system's architecture
- Every change requires touching many files across the codebase
- High defect rate, especially regression bugs
- New team members take months to become productive
- Fear of refactoring (everything is load-bearing)

**Root Cause:** No upfront design. Organic growth without architectural governance. Pressure to ship without paying down structural debt.

**Remedy:**
- Identify implicit module boundaries and make them explicit
- Introduce module dependency rules and enforce with tooling
- Strangler fig pattern: build new structure around the old
- Incremental refactoring with characterization tests as safety net

### 1.3 Golden Hammer

**Description:** Using a familiar technology or pattern for every problem, regardless of fit. "When all you have is a hammer, everything looks like a nail."

**Symptoms:**
- Same technology used where it is clearly suboptimal
- Team resists evaluating alternatives
- Workarounds to force-fit the technology to the problem
- Performance or usability issues from technology mismatch

**Root Cause:** Comfort with familiar tools. Fear of learning new technology. Past success with the tool creating overconfidence.

**Remedy:**
- Evaluate technology fit against the specific problem's forces
- Use the research workflow to objectively compare alternatives
- Accept that different problems require different tools

### 1.4 Premature Microservices

**Description:** Decomposing a system into microservices before understanding the domain boundaries, when a modular monolith would be simpler and more appropriate.

**Symptoms:**
- Team of 3-5 managing 15+ services
- Most services are trivial (100-200 lines of business logic)
- More time spent on infrastructure than features
- Service boundaries change frequently (boundaries were wrong)
- Latency and reliability worse than a single deployment

**Root Cause:** Equating microservices with modern architecture. Resume-driven development. Not understanding the costs of distribution.

**Remedy:**
- Consolidate into a modular monolith
- Extract services only when forced by scaling, deployment, or team needs
- See `os/teams/architect/templates/monolith.md`

---

## 2. Code Anti-Patterns

### 2.1 God Object / God Class

**Description:** A single class or module that knows too much and does too much. It is the center of the universe in the codebase.

**Symptoms:**
- One class with hundreds or thousands of lines
- Most other classes depend on this one
- Frequent merge conflicts in this file
- Difficult to test in isolation

**Remedy:** Decompose into smaller, focused classes using Single Responsibility Principle. Extract methods into domain services. Use dependency injection.

### 2.2 Shotgun Surgery

**Description:** A single logical change requires modifications in many different places across the codebase.

**Symptoms:**
- Adding a field to a model requires changes in 10+ files
- Business rule changes touch UI, API, service, and database layers
- High risk of missing one of the required changes

**Remedy:** Group related behavior together (vertical slice, feature modules). Use code generation for repetitive patterns. Reduce coupling.

### 2.3 Spaghetti Code

**Description:** Code with complex, tangled control flow. Deep nesting, long methods, and unclear data flow make it nearly impossible to follow.

**Symptoms:**
- Functions exceeding 100 lines
- Nesting deeper than 3-4 levels
- Multiple return points with side effects
- Goto-like control flow (exceptions used for flow control)

**Remedy:** Extract methods. Flatten nesting with early returns (guard clauses). Replace conditionals with polymorphism. Apply cyclomatic complexity limits.

### 2.4 Copy-Paste Programming

**Description:** Duplicating code instead of abstracting shared logic. Leads to inconsistent behavior when one copy is updated and others are not.

**Symptoms:**
- Same logic in multiple places with slight variations
- Bug fixes applied in one location but not others
- Inconsistent behavior across similar features

**Remedy:** Extract shared logic into reusable functions, libraries, or services. DRY principle — but only for true duplication (not coincidental similarity).

### 2.5 Stringly-Typed Code

**Description:** Using raw strings where structured types (enums, value objects, domain types) would be safer and clearer.

**Symptoms:**
- Functions accepting `string` for status, type, role, currency, etc.
- Typos in strings cause runtime failures
- No IDE autocompletion or type checking
- Validation logic scattered across the codebase

**Remedy:** Replace strings with enums, value objects, or domain types. Validate at system boundaries and use typed values internally.

---

## 3. Database Anti-Patterns

### 3.1 Entity-Attribute-Value (EAV)

**Description:** Storing structured data as generic key-value pairs in a single table instead of proper columns.

**Symptoms:**
- Table with columns: `entity_id`, `attribute_name`, `attribute_value`
- Complex queries with many self-joins to reconstruct entities
- No type safety (all values are strings)
- Cannot enforce constraints or foreign keys on attributes

**Remedy:** Use proper relational modeling. If flexibility is genuinely needed, use JSONB columns in PostgreSQL or a document database.

### 3.2 N+1 Queries

**Description:** Loading a list of N items, then executing one additional query per item to load related data.

**Symptoms:**
- Endpoint latency scales linearly with result count
- Database log shows hundreds of identical queries per request
- Performance degrades as data grows

**Remedy:** Use eager loading (JOIN), batch loading (WHERE id IN (...)), or DataLoader pattern. Detect with query counting in tests.

### 3.3 Shared Database Integration

**Description:** Multiple services reading from and writing to the same database, using the database as an integration mechanism.

**Symptoms:**
- Schema changes in one service break another service
- Cannot independently scale or deploy services
- No clear data ownership
- Difficult to reason about data consistency

**Remedy:** Database per service. Integrate through APIs or events. Use CDC for migration from shared database.

### 3.4 Premature Denormalization

**Description:** Denormalizing the database for performance before measuring whether normalization is actually a bottleneck.

**Symptoms:**
- Duplicate data across tables with synchronization bugs
- Complex update logic to keep denormalized data consistent
- Write performance degraded by maintaining redundant copies

**Remedy:** Start normalized. Measure actual query performance. Denormalize only the specific queries that are too slow, with explicit documentation of the trade-off.

---

## 4. Security Anti-Patterns

### 4.1 Security by Obscurity

**Description:** Relying on secrecy of implementation details (hidden URLs, unpublished APIs, obfuscated code) as the primary security control.

**Remedy:** Implement proper authentication and authorization. Assume attackers know everything about your system.

### 4.2 Trust the Client

**Description:** Performing validation, authorization, or security checks only on the client side, trusting that the client is honest.

**Remedy:** Validate everything server-side. Client-side validation is a UX feature, not a security control.

### 4.3 Secrets in Code

**Description:** Hardcoding passwords, API keys, tokens, or connection strings in source code, configuration files, or environment variable definitions committed to version control.

**Remedy:** Use a secrets vault. Inject secrets at runtime. Scan for secrets in CI.

### 4.4 Overly Permissive Access

**Description:** Granting broad permissions because it is easier than determining the minimum required permissions.

**Remedy:** Start with zero permissions. Add only what is needed. Review permissions quarterly.

---

## 5. Operational Anti-Patterns

### 5.1 Snowflake Servers

**Description:** Infrastructure that is manually configured and cannot be reproduced. Each server is unique, like a snowflake.

**Remedy:** Infrastructure as Code. Immutable infrastructure. Configuration management.

### 5.2 Alert Fatigue

**Description:** Too many alerts, most of which are false positives or non-actionable, causing the team to ignore all alerts.

**Remedy:** Alert on symptoms (user impact), not causes. Every alert must be actionable. Review and tune alerts monthly. Delete alerts nobody acts on.

### 5.3 Log and Pray

**Description:** Logging everything with no structure, no analysis, and no monitoring. Logs are only checked after an incident, when they are often insufficient.

**Remedy:** Structured logging. Centralized log aggregation. Proactive log-based alerting. Log level discipline.

### 5.4 YOLO Deployments

**Description:** Deploying directly to production without staging, testing, canary, or rollback capability. "It worked on my machine."

**Remedy:** CI/CD pipeline with quality gates. Staging environment. Canary deployments. Automated rollback.

---

## 6. Process Anti-Patterns

### 6.1 Cargo Cult Engineering

**Description:** Adopting practices from successful companies without understanding why those practices work. "Netflix does chaos engineering, so we should too" (with 3 users and no monitoring).

**Remedy:** Understand the problem before adopting the solution. Evaluate practices against your actual constraints and maturity level.

### 6.2 Resume-Driven Development

**Description:** Choosing technologies because they look good on a resume, not because they are the best fit for the problem.

**Remedy:** Evaluate technology against the specific problem, team skills, and business constraints. Use the research workflow.

### 6.3 Not Invented Here (NIH)

**Description:** Building custom solutions for problems that are well-solved by existing libraries, frameworks, or services.

**Remedy:** Build vs. buy analysis. Custom code is a liability. Use proven solutions unless you have a specific, documented reason to build your own.

---

*Anti-patterns are the collective memory of the industry's mistakes. Learning from them is cheaper than repeating them. When you recognize an anti-pattern in your system, fix it — do not rationalize it.*
