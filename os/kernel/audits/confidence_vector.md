# Confidence Vector Model

> A single score is a lie. Confidence requires evidence, uncertainty, and trajectory.

---

## Why Confidence Alone Is Not Enough

A system that scores 0.85 on Coherence sounds good. But what if:
- The score is based on a superficial check, not deep analysis? (Low evidence)
- The assessor is unsure whether two subsystems actually agree? (High uncertainty)
- The score dropped from 0.92 last iteration? (Negative delta)

A scalar confidence score hides these critical signals. The Confidence Vector Model replaces scalar scores with a 5-field vector that makes the quality of the assessment as visible as the assessment itself.

**The fundamental equation**: Reliable confidence = high score + high evidence + low uncertainty + stable or improving trajectory.

**The anti-pattern we prevent**: "Confidence theater" — reporting high scores with low evidence and high uncertainty. This is the most dangerous failure mode in any assessment system because it creates false assurance.

---

## The 5-Field Confidence Vector

Every audit dimension produces a vector with exactly these fields:

```
{
  "score": 0.0 - 1.0,
  "evidence": 0.0 - 1.0,
  "uncertainty": 0.0 - 1.0,
  "delta": -1.0 - +1.0,
  "status": "pass" | "warn" | "fail"
}
```

### Field 1: Score (0.0 - 1.0)

The assessed quality level for this dimension.

**Calculation**: Weighted assessment based on the measurement method defined in `universal_backbone.md` for each dimension. The scoring guide in each dimension definition provides the calibration.

**Rules**:
- Scores are never rounded up to pass a threshold. 0.749 is not 0.75.
- Scores must be justified with specific evidence, not gut feeling.
- When sub-scores exist (e.g., multiple measurement criteria), the final score is the weighted average, with weights reflecting criticality.

### Field 2: Evidence (0.0 - 1.0)

How much supporting evidence exists for the score.

**Calculation**:
- 1.0: All evidence requirements for this dimension are fully satisfied. Deep analysis completed.
- 0.8: Most evidence present. Minor gaps that do not affect confidence in the score.
- 0.6: Moderate evidence. Score is likely correct but not thoroughly validated.
- 0.4: Limited evidence. Score is an informed estimate, not a measured value.
- 0.2: Minimal evidence. Score is largely inferred from incomplete information.
- 0.0: No evidence. Score is a guess.

**Rules**:
- Evidence is assessed against the specific evidence requirements listed for each dimension in `universal_backbone.md`.
- A checklist approach works: count evidence items present / total evidence items required.
- Evidence quality matters — a superficial analysis counts less than a deep one.

### Field 3: Uncertainty (0.0 - 1.0)

How uncertain the assessor is about the score, independent of evidence level.

**Calculation**:
- 0.0: No uncertainty. The score is definitively correct.
- 0.1 - 0.2: Very low uncertainty. Minor ambiguities that would not change the score significantly.
- 0.3 - 0.4: Moderate uncertainty. Some aspects could reasonably be scored differently.
- 0.5 - 0.6: High uncertainty. The score could shift significantly with more analysis.
- 0.7+: Very high uncertainty. The score is unreliable.

**Why uncertainty is separate from evidence**: You can have high evidence and high uncertainty (conflicting evidence). You can have low evidence and low uncertainty (the situation is simple enough to assess without deep analysis). They are independent signals.

**Rules**:
- Uncertainty must reflect genuine epistemic state, not be used to hedge.
- If uncertainty is above 0.5, the assessor must explain what would reduce it.
- Uncertainty naturally decreases across iterations as more analysis is performed.

### Field 4: Delta (-1.0 to +1.0)

Change in score from the previous iteration.

**Calculation**: `delta = current_score - previous_score`

**Rules**:
- First iteration: delta = 0.0 (no previous score to compare against).
- Positive delta = improvement. Negative delta = regression.
- Delta is used to detect stagnation: if delta is near zero after a reroute, the reroute was ineffective.
- Large positive deltas after rerouting confirm the reroute target was correct.

### Field 5: Status

The pass/warn/fail determination.

**Calculation**:
```
if score >= score_threshold
   AND evidence >= evidence_threshold
   AND uncertainty <= uncertainty_max:
    status = "pass"

elif score >= score_threshold * 0.9
   AND evidence >= evidence_threshold * 0.8
   AND uncertainty <= uncertainty_max * 1.2:
    status = "warn"

else:
    status = "fail"
```

Default thresholds (overridable per mode, see `threshold_logic.md`):
- `score_threshold`: 0.75
- `evidence_threshold`: 0.6
- `uncertainty_max`: 0.3

**Rules**:
- "warn" status does not block progress but is flagged for attention.
- "fail" status triggers rerouting (see `reroute_logic.md`).
- A "pass" with borderline values should still be noted in the evolution ledger.

---

## Confidence Theater Detection

The most dangerous pattern is a vector that looks good on the surface but is hollow:

```
{
  "score": 0.88,       # Looks great
  "evidence": 0.25,    # Almost no evidence
  "uncertainty": 0.65, # Very uncertain
  "delta": 0.0,        # No change
  "status": "fail"     # Correctly caught by the model
}
```

**Detection rules**:
- `score >= 0.75 AND evidence < 0.4` = suspected confidence theater. Flag immediately.
- `score >= 0.75 AND uncertainty > 0.5` = unreliable assessment. Reroute to Phase 6 for deeper audit.
- `evidence < 0.3 AND uncertainty < 0.2` = impossible combination. Assessor is overconfident. Reject and re-audit.

---

## How Deltas Drive Rerouting

Deltas are the primary signal for detecting ineffective reroutes:

| Delta Pattern | Interpretation | Action |
|---|---|---|
| delta > +0.15 after reroute | Reroute was effective | Continue forward |
| delta +0.05 to +0.15 after reroute | Partial improvement | Allow one more iteration |
| delta < +0.05 after reroute | Reroute was ineffective | Try deeper reroute target |
| delta < 0 after reroute | Regression | Escalate — something is wrong |
| delta = 0 for 3+ iterations | Stagnation | Escalate or trigger overdrive mode |

---

## Vector Aggregation

When an overall system confidence is needed (e.g., for pass/fail at the phase level):

1. Compute vectors for all 8 dimensions.
2. Overall status = "pass" only if ALL dimensions have status "pass" or "warn" (with at most 2 warnings).
3. Overall status = "fail" if ANY dimension has status "fail".
4. Report the weakest dimension prominently — it determines the reroute target.

Do NOT average scores across dimensions. A system that scores 0.95 on 7 dimensions and 0.3 on one dimension is NOT a 0.87 system. It is a system with a critical failure in one dimension.

---

## Vector Serialization Format

For machine readability and ledger recording:

```json
{
  "dimension": "coherence",
  "vector": {
    "score": 0.82,
    "evidence": 0.75,
    "uncertainty": 0.15,
    "delta": 0.12,
    "status": "pass"
  },
  "justification": "All components trace to stated purpose. Two minor interface mismatches identified and documented.",
  "iteration": 3
}
```

All vectors must include a human-readable justification. Numbers without narrative are not acceptable.

---

*Confidence Vector Model v1.0 — Kernel Audit Infrastructure*
