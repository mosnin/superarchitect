# /build-system -- Full System Build Command

Build a complete, production-ready system from a natural language description.

## Activation Sequence

When this command is invoked with a system description, execute the following:

### Step 1: Boot the OS
1. Read `CLAUDE.md` to activate the SuperArchitect OS
2. Read `os/manifest.md` to load team and workflow registries
3. Read `os/commander/COMMANDER.md` to instantiate the Commander persona
4. Read `os/commander/dispatch.md` to load task routing logic
5. Read `os/commander/protocols.md` to load communication protocols

### Step 2: Load the Build Workflow
1. Read `os/workflows/new-system.md` to load the 7-phase build workflow
2. Confirm workflow phases are understood

### Step 3: Execute Phase 1 -- Discovery
1. Read `os/teams/product/PRODUCT.md` to activate the Product Agent
2. Read `os/teams/product/frameworks.md` for prioritization tools
3. Analyze the system description provided by the user
4. Classify the system type (SaaS, API, Data Pipeline, AI/ML, Enterprise, Tool, Mobile)
5. Produce:
   - Problem statement
   - Target user personas
   - Functional requirements (prioritized with MoSCoW)
   - Non-functional requirements with numeric targets
   - Success metrics
6. Validate Phase 1 quality gate before proceeding

### Step 4: Execute Phase 2 -- Architecture
1. Read `os/teams/architect/ARCHITECT.md` to activate the Architect Agent
2. Read `os/teams/architect/patterns.md` for pattern selection
3. Read the appropriate template:
   - `os/teams/architect/templates/microservices.md` for distributed systems
   - `os/teams/architect/templates/monolith.md` for single-team systems
   - `os/teams/architect/templates/event-driven.md` for event-based systems
4. Design the system architecture:
   - Architecture pattern selection with justification
   - Service/module decomposition
   - Technology stack selection
   - API design
   - Data model
   - Infrastructure topology
5. Produce Architecture Decision Records for key choices
6. Validate Phase 2 quality gate

### Step 5: Execute Phase 3 -- Security Architecture
1. Read `os/teams/security/SECURITY.md` to activate the Security Agent
2. Read `os/teams/security/standards.md` for security requirements
3. Perform STRIDE threat modeling on the architecture
4. Define authentication, authorization, and encryption strategy
5. Validate Phase 3 quality gate

### Step 6: Execute Phase 4 -- Implementation
1. Read `os/teams/engineer/ENGINEER.md` to activate the Engineer Agent
2. Read `os/teams/engineer/standards.md` for coding standards
3. Read `os/teams/devops/DEVOPS.md` to activate the DevOps Agent
4. Read `os/teams/devops/standards.md` for infrastructure standards
5. Implement the system:
   - Project structure and configuration
   - Domain model and business logic
   - API endpoints with validation
   - Data access layer with migrations
   - Authentication and authorization
   - Background jobs and event handlers
   - Observability (logging, metrics, tracing)
   - Infrastructure as Code
   - CI/CD pipeline
   - Dockerfiles and container configuration
6. Write tests alongside implementation
7. Validate Phase 4 quality gate

### Step 7: Execute Phase 5 -- Quality Assurance
1. Read `os/teams/qa/QA.md` to activate the QA Agent
2. Read `os/teams/qa/standards.md` for testing standards
3. Read `os/teams/qa/checklists.md` for pre-release checklist
4. Execute the pre-release checklist
5. Run all tests and verify coverage
6. Validate Phase 5 quality gate

### Step 8: Execute Phase 6 -- Documentation
1. Write README with project overview and setup instructions
2. Write API documentation
3. Write architecture documentation
4. Write deployment and operations guide
5. Validate Phase 6 quality gate

### Step 9: Execute Phase 7 -- Delivery
1. Run final validation across all artifacts
2. Verify all quality gates passed
3. Produce delivery summary
4. Present the complete system to the user

## Usage

```
/build-system A SaaS platform for managing restaurant reservations with real-time availability, customer notifications, and analytics dashboard
```

## Notes
- The build is fully autonomous. The OS makes all technical decisions.
- Escalation to human occurs only for business trade-offs or ambiguous requirements.
- Each phase must pass its quality gate before the next phase begins.
- The system produced is production-ready with tests, documentation, and infrastructure.

$ARGUMENTS
