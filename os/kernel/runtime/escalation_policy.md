# Escalation Policy

> Escalation is a signal of genuine ambiguity, not a crutch for indecision.

---

## When to Escalate

Escalation to a human operator occurs ONLY when the system encounters a situation it cannot resolve autonomously with confidence. Escalation is rare by design. If escalation is happening frequently, the system's reasoning or configuration is flawed.

### Mandatory Escalation Conditions

| Condition | Detection | Rationale |
|---|---|---|
| **Score stagnation** | 3 consecutive iterations with delta < 0.05 for any failing dimension | The system is not converging. It needs human guidance on priorities or constraints. |
| **Max reroutes exceeded** | reroute_count > max_reroute_iterations | Safety valve. The system has exhausted its correction budget. |
| **Narrow gap + high uncertainty + high consequence** | Top candidates within 0.1 score gap AND uncertainty > 0.4 AND consequence level >= "high" | The decision depends on preferences or trade-offs that are outside the technical scope. |
| **Contradictory requirements** | Phase 1 detects requirements that cannot all be satisfied simultaneously | Business must decide which requirements to relax. |
| **Regulatory ambiguity** | Security or compliance analysis encounters requirements that require legal interpretation | Engineering cannot make legal decisions. |
| **Irreversible operations** | Build plan includes destroying or replacing production resources | Explicit human approval required for destructive actions. |

### Never Escalate For

- Technology selection (the system has full authority).
- Architectural pattern choice (the system has full authority).
- Implementation details (the system has full authority).
- Test strategy (the system has full authority).
- Uncertainty that more analysis can resolve (run overdrive instead).

---

## Escalation Message Format

Every escalation produces a structured message delivered to the human operator:

```json
{
  "type": "ESCALATION",
  "severity": "high",
  "escalation_id": "ESC-<sequence>",
  "timestamp": "2026-03-29T14:32:00Z",
  "context": {
    "current_phase": "PHASE_4",
    "iteration": 5,
    "reroute_count": 3,
    "mode": "standard"
  },
  "situation": "Two candidate architectures scored within 0.08 of each other. Uncertainty is 0.45 for both. The primary differentiator is a cost vs. operational complexity trade-off that depends on organizational preference.",
  "options": [
    {
      "option": "A",
      "description": "Event-driven microservices on managed cloud services",
      "score": 0.82,
      "pros": ["Lower operational burden", "Faster time to market"],
      "cons": ["Higher cloud costs at scale ($X/month)", "Vendor lock-in risk"],
      "confidence": 0.55
    },
    {
      "option": "B",
      "description": "Modular monolith on self-managed Kubernetes",
      "score": 0.74,
      "pros": ["Lower cloud costs at scale", "Full portability"],
      "cons": ["Higher operational complexity", "Requires Kubernetes expertise"],
      "confidence": 0.55
    }
  ],
  "recommendation": "Option A, unless the organization has strong Kubernetes expertise and cost sensitivity above $X/month threshold.",
  "decision_needed": "Select architecture approach. This is a business trade-off between operational cost and operational complexity.",
  "deadline": "Blocking — build cannot proceed without this decision.",
  "resume_instructions": "Respond with option letter (A or B) and any additional constraints. The build will resume from Phase 4 with the selected candidate."
}
```

### Required Fields

| Field | Purpose |
|---|---|
| `severity` | "low" (informational), "medium" (blocking but not urgent), "high" (blocking and time-sensitive), "critical" (safety or compliance risk) |
| `situation` | Plain-language description of what happened and why the system cannot decide. |
| `options` | All viable options with scores, pros, cons, and confidence levels. Minimum 2 options. |
| `recommendation` | The system's best guess, with conditions. Never leave this blank — always provide a recommendation even when uncertain. |
| `decision_needed` | Exactly what the human must decide. One clear question. |
| `resume_instructions` | How the human should respond and what happens next. |

---

## Resume Protocol

After a human provides an escalation decision:

1. **Validate the response**: Ensure the human selected a valid option or provided a clear directive.
2. **Record the decision**: Log the human's choice in the evolution ledger as a special entry type (`"type": "HUMAN_DECISION"`).
3. **Inject the decision**: Feed the human's choice into the current phase as a constraint.
4. **Resume the loop**: The Controller Loop continues from the phase where it was blocked.
5. **Do not re-escalate the same issue**: Once a human decides, that decision is binding for the remainder of the build. If new information emerges that contradicts the decision, note it in the ledger but do not re-escalate unless the human explicitly asked to be notified.

---

## Anti-Pattern: Escalation as a Crutch

Escalation must be rare. Signs that escalation is being misused:

| Signal | Problem | Fix |
|---|---|---|
| More than 3 escalations per build | System is not autonomous enough | Review mode selection and thresholds. Tighten the escalation conditions. |
| Escalation for technology choices | System is not applying its knowledge base | Ensure tech radar and case studies are loaded. |
| Escalation with only 1 option | Not a real escalation — system already knows the answer | Remove the escalation. Proceed with the single option. |
| Escalation without a recommendation | System is abdicating responsibility | Always provide a recommendation. The human can override, but the system must have an opinion. |
| Repeated escalation for the same type of decision | Missing policy or standard | Create a standard or configuration that preempts this class of escalation. |

---

## Escalation Severity Guide

| Severity | Response Time | Examples |
|---|---|---|
| `low` | Non-blocking. FYI only. | "Chose PostgreSQL over MySQL based on feature requirements. Noting for awareness." |
| `medium` | Blocking. Respond within reasonable timeframe. | "Two deployment strategies are viable. Build can continue on either path." |
| `high` | Blocking. Respond promptly. | "Architecture candidates are too close to call. Cost vs. complexity trade-off." |
| `critical` | Blocking. Immediate attention. | "Compliance requirement may prohibit the planned approach. Legal review needed." |

Note: `low` severity escalations are informational — the system does not actually block. It continues with its recommendation and notes the escalation for human review.

---

*Escalation Policy v1.0 — Kernel Runtime Engine*
