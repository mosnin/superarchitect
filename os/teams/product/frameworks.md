# Product Frameworks — SuperArchitect OS

**Version:** 1.0
**Owner:** Product Agent (Team 9)
**Last Updated:** 2026-03-29

---

## Purpose

This document contains the product frameworks used by the Product Agent for structured decision-making. Each framework includes its purpose, when to use it, how to apply it, and its limitations.

---

## 1. Prioritization Frameworks

### 1.1 RICE Scoring

**Purpose:** Quantitatively prioritize features based on expected impact.

**Components:**
| Factor | Definition | Scale |
|--------|-----------|-------|
| Reach | How many users affected per quarter | Absolute number |
| Impact | Effect on each user | 3=massive, 2=high, 1=medium, 0.5=low, 0.25=minimal |
| Confidence | How sure are you about estimates | 100%=high, 80%=medium, 50%=low |
| Effort | Person-months to implement | Absolute number |

**Formula:** `RICE Score = (Reach * Impact * Confidence) / Effort`

**When to use:** Comparing features across different domains when you need an objective ranking.

**Limitations:** Garbage in, garbage out. If Reach and Impact estimates are guesses, the score is meaningless. Use Confidence to discount uncertainty.

### 1.2 MoSCoW Method

**Purpose:** Classify requirements by necessity for a specific release.

| Category | Definition | Rule |
|----------|-----------|------|
| Must Have | Non-negotiable. Release fails without it. | If you remove it and the release is unusable, it is Must Have |
| Should Have | Important but not critical. Painful to omit. | Would significantly reduce value if excluded |
| Could Have | Desirable. Include if time/budget permits. | Nice to have, no significant impact if excluded |
| Won't Have | Out of scope for this release. May do later. | Explicitly excluded to manage scope |

**When to use:** Scoping a specific release or MVP. Forces hard decisions about what is truly essential.

**Rules:**
- Must Have should be no more than 60% of estimated effort
- If everything is Must Have, you have not prioritized

### 1.3 Kano Model

**Purpose:** Classify features by their relationship to user satisfaction.

| Category | Description | Investment Strategy |
|----------|-------------|-------------------|
| Basic (Must-be) | Expected by default. Absence causes dissatisfaction. Presence does not delight. | Implement fully. No competitive advantage but absence is a deal-breaker. |
| Performance (One-dimensional) | More is better. Linear relationship to satisfaction. | Invest proportionally. Competitive differentiation. |
| Excitement (Attractive) | Unexpected delighters. Absence does not disappoint but presence creates loyalty. | Invest selectively. High impact on brand and word-of-mouth. |
| Indifferent | Users do not care either way. | Do not build. Waste of resources. |
| Reverse | Some users actively dislike this feature. | Avoid or make optional. |

**When to use:** Understanding which features to invest in for maximum user satisfaction.

### 1.4 Weighted Shortest Job First (WSJF)

**Purpose:** Maximize value delivery in flow-based (Kanban/SAFe) environments.

**Formula:** `WSJF = Cost of Delay / Job Duration`

**Cost of Delay = User/Business Value + Time Criticality + Risk Reduction/Opportunity Enablement**

Each component scored on Fibonacci scale: 1, 2, 3, 5, 8, 13, 20.

**When to use:** Ordering a backlog when items have different time sensitivities.

### 1.5 Opportunity Scoring

**Purpose:** Find underserved user needs with the highest improvement potential.

**Process:**
1. List the jobs users are trying to do
2. Survey users: "How important is this?" (1-10)
3. Survey users: "How satisfied are you with current solutions?" (1-10)
4. Calculate: `Opportunity = Importance + max(Importance - Satisfaction, 0)`
5. Prioritize: high importance + low satisfaction = biggest opportunity

---

## 2. Strategy Frameworks

### 2.1 Jobs to Be Done (JTBD)

**Purpose:** Understand what users are really trying to accomplish, independent of your product.

**Job Statement Format:**
```
When [situation], I want to [motivation], so I can [expected outcome].
```

**Process:**
1. Interview users about their workflow (not your product)
2. Identify the functional job (what they are trying to do)
3. Identify the emotional job (how they want to feel)
4. Identify the social job (how they want to be perceived)
5. Map competing solutions (what they use today, including non-consumption)
6. Find the switch triggers (what would make them change solutions)

**Outcome:** Feature ideas grounded in real user needs, not feature requests.

### 2.2 Product Vision Canvas

| Element | Question |
|---------|----------|
| Target Group | Who are we building this for? |
| Needs | What problem are we solving for them? |
| Product | What are we building? (one sentence) |
| Business Goals | Why is the business building this? |
| Competitors | What alternatives exist today? |
| Differentiators | Why will users choose this over alternatives? |
| Revenue Model | How does this generate value for the business? |
| Key Metrics | How will we measure success? |

### 2.3 Lean Canvas

A one-page business model for new products or features:

| Column | Left Side | Right Side |
|--------|-----------|------------|
| Row 1 | Problem (top 3) | Solution (top 3 features) |
| Row 2 | Key Metrics | Unique Value Proposition |
| Row 3 | Channels | Unfair Advantage |
| Row 4 | Cost Structure | Revenue Streams |
| Center | Customer Segments | Early Adopters |

**When to use:** Validating a new product idea or major feature bet.

---

## 3. Discovery Frameworks

### 3.1 Assumption Mapping

**Purpose:** Surface and prioritize the riskiest assumptions before building.

**Process:**
1. List all assumptions about: users, problem, solution, business model
2. Plot on a 2x2 matrix: Known/Unknown vs. Evidence/No Evidence
3. Prioritize: Unknown + No Evidence = highest risk
4. Design experiments to validate riskiest assumptions first
5. Kill ideas where critical assumptions are invalidated

### 3.2 Experiment Design

**Hypothesis Format:**
```
We believe [this capability]
Will result in [this outcome]
We will know we are right when [measurable signal]
```

**Experiment Types (ordered by cost and confidence):**
| Type | Cost | Confidence | Duration |
|------|------|-----------|----------|
| Desk research | Very low | Low | Hours |
| User interviews | Low | Medium | Days |
| Survey | Low | Medium | Days |
| Fake door test | Low | Medium | Days |
| Prototype test | Medium | Medium-High | 1-2 weeks |
| Concierge MVP | Medium | High | 2-4 weeks |
| A/B test | High | Very High | 2-4 weeks |
| Full build | Very High | Highest | Weeks-months |

**Rule:** Always start with the cheapest experiment that can disprove your hypothesis. Only escalate to more expensive experiments when cheaper ones produce inconclusive results.

### 3.3 Design Sprint (Compressed)

**Day 1: Map and Target**
- Map the user journey
- Choose a target area to focus on
- Define sprint questions (what must be true for this to succeed?)

**Day 2: Sketch Solutions**
- Individual solution sketching (diverge)
- Lightning demos of competitor/analogous solutions
- Crazy 8s rapid ideation

**Day 3: Decide and Prototype**
- Vote on best solution components
- Create storyboard for prototype
- Build high-fidelity prototype

**Day 4: Test**
- Test prototype with 5 target users
- Document patterns in feedback
- Decide: build, iterate, or kill

---

## 4. Measurement Frameworks

### 4.1 HEART Framework (Google)

| Dimension | Definition | Example Metrics |
|-----------|-----------|----------------|
| Happiness | User satisfaction and sentiment | NPS, CSAT, SUS score |
| Engagement | Depth of product usage | DAU/MAU, session duration, feature adoption |
| Adoption | New users starting to use the product | Sign-up rate, onboarding completion |
| Retention | Users continuing to use the product | D7/D30 retention, churn rate |
| Task Success | Users completing intended tasks | Task completion rate, time on task, error rate |

**Process:**
1. Choose the most relevant HEART dimensions for your product
2. Define Goals for each dimension
3. Define Signals (user actions that indicate goal progress)
4. Define Metrics (measurable quantities from signals)

### 4.2 Pirate Metrics (AARRR)

| Stage | Question | Key Metrics |
|-------|----------|------------|
| Acquisition | How do users find us? | Traffic, channel effectiveness, CAC |
| Activation | Do users have a good first experience? | Onboarding completion, time to first value |
| Retention | Do users come back? | D1/D7/D30 retention, churn |
| Referral | Do users tell others? | NPS, viral coefficient, invite rate |
| Revenue | Do users pay? | MRR, ARPU, LTV, conversion rate |

### 4.3 North Star Framework

**Structure:**
```
North Star Metric
├── Input Metric 1 (breadth: how many users)
├── Input Metric 2 (depth: how much they use it)
├── Input Metric 3 (frequency: how often they use it)
└── Input Metric 4 (efficiency: how quickly they get value)
```

**North Star Metric Selection Criteria:**
1. Expresses the core value your product delivers
2. Is a leading indicator of revenue
3. Is measurable and reportable on a weekly cadence
4. Is actionable by the product team
5. Is understandable by the entire organization

---

## 5. Communication Frameworks

### 5.1 PRD (Product Requirements Document) Template

```markdown
# [Feature Name] PRD

## Problem Statement
[Who has this problem? What is the problem? Why does it matter? How big is it?]

## Goals and Non-Goals
### Goals
- [Measurable outcome 1]
- [Measurable outcome 2]
### Non-Goals
- [What we are explicitly NOT doing]

## User Stories
[User stories with acceptance criteria]

## Success Metrics
| Metric | Current | Target | Timeline |
|--------|---------|--------|----------|

## Design
[Wireframes, prototypes, user flows — link to design artifacts]

## Technical Considerations
[Constraints, dependencies, technical risks — from Architecture team]

## Launch Plan
[Rollout strategy, feature flags, A/B test design]

## Open Questions
[Unresolved decisions requiring input]
```

### 5.2 Decision Log Template

| Date | Decision | Options Considered | Rationale | Owner | Reversible? |
|------|---------|-------------------|-----------|-------|-------------|

---

## 6. Anti-Patterns to Avoid

| Anti-Pattern | Description | Alternative |
|-------------|-------------|-------------|
| Feature Parity | Building features because competitors have them | Solve your users' actual problems |
| HiPPO | Highest Paid Person's Opinion drives decisions | Data + frameworks drive decisions |
| Build Trap | Measuring success by output (features shipped) | Measure outcomes (user behavior change) |
| Confirmation Bias | Looking for data that supports your hypothesis | Actively seek disconfirming evidence |
| Scope Creep | Requirements expand continuously | MoSCoW + firm release scope |
| Goldplating | Adding polish to features nobody uses | Ship fast, iterate on what matters |
| Sunk Cost | Continuing because of past investment | Evaluate based on future value only |

---

*Frameworks are thinking tools, not bureaucratic overhead. Use the simplest framework that produces the clarity you need. Do not apply every framework to every decision.*
