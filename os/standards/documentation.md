# Documentation Standards — SuperArchitect OS

**Version:** 1.0
**Owner:** SuperArchitect OS Core Team
**Last Updated:** 2026-03-28
**Audience:** All agents and engineers operating within the SuperArchitect OS

---

## Documentation Philosophy

Documentation is a product. It has users with jobs to do. It has requirements. It has quality criteria. It can succeed or fail at its purpose.

Bad documentation is worse than no documentation. Inaccurate documentation misleads. Outdated documentation creates false confidence. Incomplete documentation leaves users stranded at the worst possible moment.

The SuperArchitect OS treats documentation with the same rigor as code:
- It is written before or alongside the thing it describes, not after
- It is reviewed, not just rubber-stamped
- It is tested: users try to accomplish tasks with it
- It is maintained: it changes when the system changes
- It is owned: someone is accountable for its accuracy

**The single question every documentation decision answers: "What does the reader need to accomplish, and does this document help them accomplish it?"**

If the answer is no, the document should not exist in its current form.

---

## Documentation Types

### Architecture Documentation

**Purpose:** Enable engineers to understand how the system is structured, why it was built that way, and how to make decisions that fit the design.

**Contents:** System context, component decomposition, integration diagrams, key architectural decisions (ADRs), fitness functions, technology stack rationale.

**Audience:** Engineers joining the team, architects making cross-system decisions, on-call engineers diagnosing complex failures.

**Freshness requirement:** Updated at every architectural change, reviewed quarterly.

---

### API Documentation

**Purpose:** Enable developers to integrate with the system without asking questions.

**Contents:** All endpoints with request/response schemas, authentication method, error codes, rate limits, versioning policy, working code examples in at least two languages.

**Audience:** External developers, other service teams, SDK authors.

**Freshness requirement:** Updated with every API change. Outdated API docs are a production defect.

---

### Runbooks

**Purpose:** Enable on-call engineers to diagnose and resolve operational issues without expert help.

**Contents:** Symptom description, diagnostic steps, resolution procedures, escalation path, links to dashboards and logs.

**Audience:** On-call engineers, often under stress, often unfamiliar with the specific service.

**Freshness requirement:** Updated after every incident. Tested during game days.

---

### Onboarding Documentation

**Purpose:** Enable a new team member to become productive within 2 days.

**Contents:** Repository overview, how to run the system locally, how to run tests, how to deploy, key architectural decisions and why they exist, team norms, first PR checklist.

**Audience:** Engineers joining the team for the first time.

**Freshness requirement:** Reviewed every 3 months. Any friction point reported during onboarding is fixed within 1 sprint.

---

### Decision Records (ADRs)

**Purpose:** Preserve the reasoning behind architectural decisions so future maintainers understand why the system is the way it is.

**Contents:** See Architecture Standards — ADR Template.

**Audience:** Engineers making future decisions that might conflict with or build upon past decisions.

**Freshness requirement:** Append-only. Old ADRs are superseded, not edited.

---

### Post-mortems

**Purpose:** Convert production incidents into organizational learning that prevents recurrence.

**Contents:** Incident summary, timeline, root cause analysis, impact assessment, contributing factors, lessons learned, action items with owners and due dates.

**Audience:** The entire engineering organization.

**Freshness requirement:** Draft within 24 hours of incident resolution. Final version within 5 business days.

---

## Documentation Structure Standards

Every document in the SuperArchitect OS must have the following header:

```markdown
# [Title]

**Version:** [x.y]
**Owner:** [Person or team accountable for accuracy]
**Last Updated:** [YYYY-MM-DD]
**Audience:** [Who this document is written for]
**Review Frequency:** [How often this should be reviewed]
```

Every document must also include:

- **Purpose section**: One paragraph explaining what this document is for and what the reader will be able to do after reading it
- **Table of contents** for documents longer than 500 words
- **Prerequisites**: What the reader must already know or have access to
- **Related documents**: Links to closely related documentation

---

## Writing Standards

### Clarity

Write for a reader who is intelligent but unfamiliar with the specifics of this system. Assume competence, not context. Define every acronym and domain term on first use. Never assume the reader knows what you know.

### Brevity

Every sentence must justify its existence. If a sentence does not add meaning, delete it. Paragraphs should be 3–5 sentences. If a section runs more than 300 words without a code example or diagram, ask whether it can be condensed.

### Precision

Vague language creates confusion. Avoid: "usually," "generally," "in most cases," "it depends." If it depends, say what it depends on. If there are exceptions, list them.

### Active Voice

| Passive (avoid) | Active (prefer) |
|---|---|
| "Requests are authenticated by the gateway" | "The gateway authenticates requests" |
| "Errors should be handled by the caller" | "The caller must handle errors" |
| "The database is queried by the service" | "The service queries the database" |

### Examples First

Every concept that can be illustrated with an example must have an example. Lead with the working code, then explain it. Readers learn faster from examples than from abstract descriptions.

### Formatting Conventions

- **Bold** for key terms, warnings, and important constraints
- `Code font` for all code, file paths, environment variables, and commands
- Headers are hierarchical and descriptive, not clever
- Bullet lists for unordered items; numbered lists for sequential steps
- Tables for comparative information
- No emoji in technical documentation (they do not render consistently and add noise)
- Line length ≤ 100 characters in markdown source for readability in code review

---

## The C4 Documentation Model

The C4 model documents architecture at four levels of zoom. Every system in the SuperArchitect OS must have at least Level 1 and Level 2 documented. Levels 3 and 4 are required for complex systems.

### Level 1: System Context Diagram

**What it shows:** The system and its relationships with users and other systems.

**What it answers:** "What does this system do and who uses it?"

**Markdown representation:**
```markdown
## System Context

**[System Name]** serves [user types] by [core function].

### External Actors
| Actor | Type | Interaction |
|-------|------|-------------|
| End User | Human | Uses the web UI to [action] |
| Payment Provider | External System | Receives payment requests via REST API |
| Email Service | External System | Receives transactional email requests |

### Context Diagram
[Link to or embed diagram image]
```

### Level 2: Container Diagram

**What it shows:** The high-level technical components (applications, databases, services) and how they communicate.

**What it answers:** "What are the deployable units and how do they talk to each other?"

**Markdown representation:**
```markdown
## Containers

| Container | Technology | Responsibility |
|-----------|------------|----------------|
| Web App | React 18 | User interface |
| API Server | Node.js / Express | Business logic, REST API |
| Worker | Node.js | Background job processing |
| Database | PostgreSQL 15 | Persistent data storage |
| Cache | Redis 7 | Session storage, rate limiting |
| Queue | RabbitMQ | Async job queue |

### Communication
- Web App → API Server: HTTPS REST
- API Server → Database: TCP (pg driver)
- API Server → Queue: AMQP
- Worker → Queue: AMQP
- API Server → Cache: Redis protocol
```

### Level 3: Component Diagram

**What it shows:** The components inside a single container — modules, services, classes — and their relationships.

**What it answers:** "What is inside this service and how is it structured?"

### Level 4: Code Diagram

**What it shows:** The implementation of a specific component — class structure, function relationships.

**What it answers:** "How exactly is this component implemented?"

**Note:** Level 4 is expensive to maintain. Only produce it for the most complex or critical components.

---

## API Documentation Standard

### OpenAPI Specification Requirements

All REST APIs must have an OpenAPI 3.1 specification file:
- Located at: `/docs/api/openapi.yaml`
- Served at: `GET /api-docs` (development) and `GET /openapi.json` (production)
- Generated from code annotations where possible (reduces drift)

**Required fields for every endpoint:**
- `summary`: One-line description of what the endpoint does
- `description`: Multi-line description with context, constraints, and behavior details
- `operationId`: Unique, meaningful identifier (e.g., `createOrder`, `listUserSessions`)
- `tags`: Category grouping for documentation navigation
- `requestBody`: Full schema with descriptions and examples for all fields
- `responses`: All possible response codes with schemas and examples
- `security`: Authentication requirements

**Required for every schema field:**
- `description`: What the field represents
- `example`: A realistic example value
- `format`: Specific format (e.g., `date-time`, `email`, `uuid`)

### Example-First Documentation

Document APIs with working, realistic examples as the primary content. Explain behavior in terms of what the example shows.

```markdown
## Create Order

### Request
POST /api/v1/orders

\`\`\`json
{
  "customer_id": "cust_01HX4MZQP1K...",
  "items": [
    { "product_id": "prod_01HX...", "quantity": 2 }
  ],
  "shipping_address": {
    "line1": "123 Main St",
    "city": "Springfield",
    "state": "IL",
    "zip": "62701",
    "country": "US"
  }
}
\`\`\`

### Response (201 Created)
\`\`\`json
{
  "order_id": "ord_01HX4MZQP1K...",
  "status": "pending",
  "total_amount_cents": 4998,
  "created_at": "2026-03-28T14:30:00Z"
}
\`\`\`

### Error Responses
| Status | Code | Description |
|--------|------|-------------|
| 400 | invalid_product | One or more product IDs do not exist |
| 400 | insufficient_stock | Requested quantity exceeds available stock |
| 422 | invalid_address | Shipping address failed validation |
```

---

## Runbook Standard

### Required Sections

Every runbook must contain:

```markdown
# Runbook: [Alert Name or Issue Title]

**Service:** [Service name]
**Alert:** [Exact alert name that triggers this runbook]
**Severity:** [P1 / P2 / P3]
**Owner:** [Team responsible]
**Last Tested:** [YYYY-MM-DD]

## Summary
One paragraph: what is this runbook for, what does it help you fix?

## Symptoms
- [Specific observable symptom 1]
- [Specific observable symptom 2]

## Impact
[Who is affected and how? What features are degraded or unavailable?]

## Diagnostic Steps
1. [First thing to check, with exact commands]
2. [Second thing to check]
3. [Decision point: if X, go to Resolution A; if Y, go to Resolution B]

## Resolution Procedures

### Resolution A: [Name]
1. [Step with exact command]
   \`\`\`bash
   kubectl rollout restart deployment/order-service -n production
   \`\`\`
2. [Verify step: how do you know it worked?]

### Resolution B: [Name]
[...]

## Escalation
If unresolved after 30 minutes: page [team] via [channel/tool].
Include: [what information to include in escalation]

## Related Resources
- Dashboard: [link]
- Logs: [link with pre-built query]
- Recent changes: [link]
- Related ADRs: [links]
```

### Automation Ratio Target

- **Target:** ≥ 60% of runbook steps are fully automated (single command to execute)
- **Minimum:** Every runbook has at least one automation entry point
- Diagnostic queries and dashboard links count toward automation ratio
- Runbooks that exceed 20 manual steps must be refactored or automated

---

## Post-Mortem Template

```markdown
# Post-Mortem: [Incident Title]

**Incident ID:** INC-[number]
**Date:** [YYYY-MM-DD]
**Duration:** [HH:MM from first alert to resolution]
**Severity:** [P1 / P2 / P3]
**Author:** [Who wrote this post-mortem]
**Status:** [Draft / Review / Final]

## Incident Summary
[2–3 sentences: what happened, who was affected, and how it was resolved.
Readable by someone who wasn't involved and needs context quickly.]

## Impact
- Users affected: [number or percentage]
- Revenue impact: [estimate if applicable]
- Features degraded: [list]
- SLO impact: [minutes of SLO violation]

## Timeline

| Time (UTC) | Event |
|------------|-------|
| HH:MM | First alert fired |
| HH:MM | On-call acknowledged |
| HH:MM | Initial diagnosis |
| HH:MM | Mitigation applied |
| HH:MM | Service restored |
| HH:MM | Root cause confirmed |
| HH:MM | Post-mortem initiated |

## Root Cause Analysis

[Precise technical description of what failed and why. Use the 5 Whys or
a fault tree to trace the causal chain to its root. Do not stop at
"the server crashed" — explain why the server crashed.]

**Root Cause:**
[One clear sentence stating the ultimate cause]

**Contributing Factors:**
1. [Factor 1]
2. [Factor 2]

## What Went Well
- [Thing that worked as intended]
- [Monitoring that fired correctly]
- [Process that helped recovery]

## What Could Be Improved
- [Detection gap]
- [Response process that could be faster]
- [Tooling that was missing]

## Lessons Learned
[Key insights that change how the team thinks about operating this system]

## Action Items

| Action | Owner | Due Date | Status |
|--------|-------|----------|--------|
| [Specific action to prevent recurrence] | [Name] | [Date] | Open |
| [Monitoring improvement] | [Name] | [Date] | Open |
| [Runbook update] | [Name] | [Date] | Open |

## Appendix
[Raw logs, screenshots, graphs from the incident. Not required in the main body.]
```

---

## Documentation Fitness

Documentation quality is measured across four dimensions. Each dimension has a target and a way to measure it.

### Findability

**Target:** Any engineer can find the relevant documentation within 60 seconds of knowing what they're looking for.

**Measurement:**
- Documentation is indexed in the team's search system
- All documents are correctly categorized and tagged
- Cross-references exist between related documents
- Test: ask a new team member to find 5 documents they've never seen; measure time

### Accuracy

**Target:** Every statement in every document is factually correct as of the last-updated date.

**Measurement:**
- Post-incident audit: did the runbook accurately describe the system?
- Periodic accuracy review: compare documentation against actual system behavior
- Friction log: track when engineers find inaccurate documentation

### Completeness

**Target:** The target audience can complete their job using the documentation without external help.

**Measurement:**
- User testing: can a new engineer complete the onboarding checklist using only the docs?
- Support ticket analysis: are support questions answered by existing docs?
- API adoption rate: can external developers self-serve from API docs?

### Currency

**Target:** All documents updated within 30 days of any change they describe.

**Measurement:**
- Automated check: flag documents with last-updated > 90 days for review
- PR checklist: every code PR that changes behavior requires a documentation update
- Stale doc count tracked as a quality metric

---

## README Standard

Every repository managed under the SuperArchitect OS must have a README.md at its root containing all of the following:

```markdown
# [Service / Library Name]

[One paragraph describing what this does and why it exists.]

## Status
[Production / Beta / Experimental] | [Build badge] | [Coverage badge]

## Quick Start
[5 lines or fewer to get a "hello world" running locally]

## Contents
[What is in this repository]

## Prerequisites
[What must be installed/configured before the Quick Start works]

## Local Development
[How to run the service locally]
[How to run tests]
[How to lint/format code]

## Configuration
[All environment variables, with description, type, default, and required/optional]

| Variable | Description | Default | Required |
|----------|-------------|---------|----------|
| DATABASE_URL | PostgreSQL connection string | none | Yes |

## API / Usage
[For services: link to API docs]
[For libraries: key usage examples]

## Architecture
[Link to architecture documentation]
[2–3 sentence overview of the key design decisions]

## Deployment
[How this service is deployed]
[Link to runbooks for common operational tasks]

## Contributing
[How to contribute: branch naming, PR process, test requirements]

## Ownership
[Team name, Slack channel, on-call rotation]
[Link to full documentation]
```
