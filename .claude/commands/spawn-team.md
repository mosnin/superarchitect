# /spawn-team -- Spawn Specialist Team Command

Activate a single specialist team for a focused task, bypassing full workflow orchestration.

## Activation Sequence

### Step 1: Boot the OS
1. Read `CLAUDE.md` to activate the SuperArchitect OS
2. Read `os/manifest.md` to load team registry
3. Read `os/commander/COMMANDER.md` for dispatch logic

### Step 2: Identify the Target Team
Parse the team name from the command arguments and map to the corresponding team file:

| Team Name | Team File | Specialization |
|-----------|-----------|---------------|
| `architect` | `os/teams/architect/ARCHITECT.md` | System design, architecture patterns, ADRs |
| `engineer` | `os/teams/engineer/ENGINEER.md` | Backend/frontend implementation, coding |
| `security` | `os/teams/security/SECURITY.md` | Threat modeling, security audits, compliance |
| `qa` | `os/teams/qa/QA.md` | Testing strategy, quality assurance, checklists |
| `devops` | `os/teams/devops/DEVOPS.md` | Infrastructure, CI/CD, deployment, monitoring |
| `data` | `os/teams/data/DATA.md` | Data modeling, pipelines, analytics |
| `designer` | `os/teams/designer/DESIGNER.md` | UI/UX design, design systems, accessibility |
| `product` | `os/teams/product/PRODUCT.md` | Requirements, prioritization, metrics |
| `researcher` | `os/teams/researcher/RESEARCHER.md` | Technology evaluation, knowledge synthesis |
| `reviewer` | `os/teams/reviewer/REVIEWER.md` | Code review, quality analysis |

### Step 3: Activate the Team
1. Read the team's primary file to instantiate the agent persona
2. Read the team's supporting files (standards, templates, checklists)
3. Apply the task description to the team's competencies
4. Execute the task according to the team's protocols

### Step 4: Deliver Results
1. Team produces artifacts according to its output protocol
2. Apply self-review against applicable standards
3. Present results to the user

## Usage

```
/spawn-team architect Design a microservices architecture for a payment processing system

/spawn-team security Perform a threat model analysis of the current authentication system

/spawn-team qa Create a comprehensive test strategy for the API layer

/spawn-team devops Design a CI/CD pipeline for a Node.js monorepo

/spawn-team product Write user stories for a notification preferences feature

/spawn-team researcher Evaluate PostgreSQL vs CockroachDB for multi-region deployment
```

## Notes
- Only one team is activated per invocation
- The team has access to the full OS knowledge base (standards, patterns, anti-patterns)
- For multi-team coordination, use `/build-system` or `/audit-system` instead
- Teams follow all OS principles and standards regardless of spawn method

$ARGUMENTS
