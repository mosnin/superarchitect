# Threshold Logic

> Thresholds determine what "good enough" means. They must be principled, not arbitrary.

---

## Universal Defaults

These defaults apply when no mode override or project-specific adjustment is present:

```json
{
  "score_threshold": 0.75,
  "evidence_threshold": 0.60,
  "uncertainty_max": 0.30
}
```

**Rationale**:
- **0.75 score**: A system that meets 75% of a dimension's criteria is structurally sound. Below this, there are material gaps.
- **0.60 evidence**: At least 60% of the evidence requirements must be met. Below this, the score is not trustworthy.
- **0.30 uncertainty**: Uncertainty above 30% means the assessment could swing enough to change the pass/fail result.

---

## Per-Mode Overrides

Each runtime mode (see `modes.md`) overrides the defaults:

| Threshold | Standard | Search Heavy | Conservative |
|---|---|---|---|
| `score_threshold` | 0.75 | 0.75 | 0.85 |
| `evidence_threshold` | 0.60 | 0.65 | 0.75 |
| `uncertainty_max` | 0.30 | 0.30 | 0.20 |

**Overdrive** inherits the thresholds of whatever mode it overlays. It does not change thresholds — it changes the depth of analysis to meet them.

---

## Project-Derived Threshold Adjustments

The Success Model (Phase 2) can define project-specific threshold adjustments. These represent the system's own quality requirements as derived from its purpose and constraints.

**How it works**:

1. Phase 2 produces a success model that includes pass/fail criteria for the system.
2. The Controller extracts any quality dimension priorities from the success model.
3. Priorities map to threshold adjustments:

```
FUNCTION derive_thresholds(success_model, mode_defaults):
  thresholds = copy(mode_defaults)

  FOR each priority IN success_model.quality_priorities:
    dimension = map_priority_to_dimension(priority)

    IF priority.level == "critical":
      thresholds[dimension].score_threshold = max(mode_defaults.score + 0.10, 0.90)
      thresholds[dimension].evidence_threshold = max(mode_defaults.evidence + 0.10, 0.80)
      thresholds[dimension].uncertainty_max = min(mode_defaults.uncertainty - 0.10, 0.15)

    ELIF priority.level == "high":
      thresholds[dimension].score_threshold = max(mode_defaults.score + 0.05, 0.85)
      thresholds[dimension].evidence_threshold = max(mode_defaults.evidence + 0.05, 0.70)

    ELIF priority.level == "relaxed":
      thresholds[dimension].score_threshold = max(mode_defaults.score - 0.05, 0.65)
      # Evidence and uncertainty thresholds are never relaxed

  RETURN thresholds
```

**Example**: A financial system's success model states "data integrity is critical." This maps to the Internal Consistency dimension, which gets elevated thresholds:
- score_threshold: 0.85 (Standard default 0.75 + 0.10)
- evidence_threshold: 0.70 (Standard default 0.60 + 0.10)
- uncertainty_max: 0.20 (Standard default 0.30 - 0.10)

---

## Dimension-Specific Thresholds

Some dimensions may warrant permanently different thresholds based on the system type, independent of the success model:

```json
{
  "dimension_overrides": {
    "failure_awareness": {
      "note": "Failure awareness is critical for any distributed system",
      "applies_when": "system has 2+ services or external dependencies",
      "score_threshold_adjustment": "+0.05"
    },
    "implementability": {
      "note": "Implementability must always pass — an unimplementable design has zero value",
      "applies_when": "always",
      "minimum_score": 0.70,
      "never_relaxed": true
    },
    "legibility": {
      "note": "Legibility can be slightly relaxed for internal tools with small teams",
      "applies_when": "internal tool AND team_size <= 5",
      "score_threshold_adjustment": "-0.05"
    }
  }
}
```

**Override priority** (highest wins):
1. Project-derived adjustments (from success model) — most specific.
2. Dimension-specific overrides — based on system characteristics.
3. Mode overrides — based on runtime mode.
4. Universal defaults — baseline.

---

## Status Determination Logic

The complete status determination for a single dimension:

```
FUNCTION determine_status(dimension, vector, thresholds):

  t = resolve_thresholds(dimension, thresholds)
  # t now has the final score_threshold, evidence_threshold, uncertainty_max
  # after applying all overrides in priority order

  IF vector.score >= t.score_threshold
     AND vector.evidence >= t.evidence_threshold
     AND vector.uncertainty <= t.uncertainty_max:
    RETURN "pass"

  # Warn zone: within 10% of passing on all criteria
  IF vector.score >= t.score_threshold * 0.90
     AND vector.evidence >= t.evidence_threshold * 0.80
     AND vector.uncertainty <= t.uncertainty_max * 1.20:
    RETURN "warn"

  RETURN "fail"
```

### Overall System Status

```
FUNCTION determine_overall_status(dimension_statuses):

  fail_count = count(status == "fail" FOR status IN dimension_statuses)
  warn_count = count(status == "warn" FOR status IN dimension_statuses)

  IF fail_count > 0:
    RETURN "fail"

  IF warn_count > 2:
    RETURN "fail"  # Too many warnings = systemic weakness

  IF warn_count > 0:
    RETURN "warn"

  RETURN "pass"
```

**Rule**: More than 2 warnings is treated as a fail. This prevents "death by a thousand cuts" where no single dimension fails but the system is weak across multiple dimensions.

---

## Threshold Immutability During Audit

Once the Controller enters an audit cycle (Phase 6), thresholds are frozen for that cycle. They cannot change mid-audit. This prevents gaming where thresholds are adjusted to make a failing system pass.

Thresholds CAN change between audit cycles (e.g., if the mode switches or the success model is refined during a reroute).

---

*Threshold Logic v1.0 — Kernel Runtime Engine*
