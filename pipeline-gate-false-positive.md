# Solution — Pipeline Gate False Positive on Unweighted Metadata

**Date:** April 3, 2026
**Originating build:** PROD30 — Atomic Deploy Pipeline
**Failure class:** Validation check tested symbol existence instead of real system state
**Status:** Active — rule applied in all subsequent validation builds

---

## Symptom

The PROD30 atomic pipeline flagged NYC as non-compliant with the recency-ban rule and halted before deploy. The deploy report named `score_recency` as a present recency component.

Investigation showed the NYC YAML config contained a `score_recency` key, but the key was metadata — preserved from a prior scoring model, present in the config for historical reference, and **unweighted**. It contributed zero to the composite score. NYC was not in fact non-compliant. The recency-ban rule was intended to catch weighted recency components in the scoring composite, not the existence of any key containing the string "recency."

The pipeline gate produced a false positive. NYC was blocked from deploying a legitimate build.

## Root cause

The validation check at Step 3 of the pipeline was implemented as field-existence:

```python
# REJECTED — field existence check
def check_no_recency(config):
    for key in config.keys():
        if "recency" in key.lower():
            raise ValidationError(f"recency-type component found: {key}")
```

The check interprets the recency-ban rule as "no field whose name contains 'recency.'" The rule as intended is "no recency-named component contributes non-zero weight to the scoring composite." The difference is the difference between checking symbols and checking behavior.

The field-existence check is brittle by design: any config that retains historical metadata, any legacy key not yet removed, any field with a name overlap produces a false positive. Worse, it produces false confidence when the check passes — a config could contain a recency component with a different name and pass the gate.

## Rule

**Pipeline validation checks the weighted composite, not field existence. Validation must test real system behavior, not surface-level symbols.**

Implementation pattern:

```python
# REQUIRED — weighted composite check
def check_no_recency(config):
    composite = config.get("scoring_composite", {})
    recency_components = {
        k: v for k, v in composite.items()
        if "recency" in k.lower()
    }
    weighted_recency = sum(
        v.get("weight", 0) for v in recency_components.values()
    )
    if weighted_recency > 0:
        raise ValidationError(
            f"recency components contribute non-zero weight "
            f"({weighted_recency}) to composite: "
            f"{list(recency_components.keys())}"
        )
```

This check reflects the actual rule: a config can contain metadata keys with "recency" in their name as long as those keys are not weighted into the composite. The check fails only when a recency component actually affects the score.

## Verification assertions (required in every subsequent validation spec)

- [ ] Validation checks test weighted behavior, not symbol existence
- [ ] Unweighted metadata keys (preserved for historical reference) do not trigger validation failures
- [ ] Each validation check includes a regression test with a known false-positive input and a known true-positive input
- [ ] False-positive regression tests are named explicitly (e.g., `test_unweighted_metadata_passes_recency_check`)

## Affected files

- `scripts/pipeline/validation.py` — primary site of the regression
- `scripts/pipeline/steps/step_3_validate_config.py` — check rewritten to test weighted composite
- `tests/pipeline/test_validation.py` — added false-positive regression tests
- `config/nyc.yml` — no change required; config was never non-compliant
- `docs/solutions/pipeline-gate-false-positive.md` — this file

## Related patterns

- [`hardcoded-filenames.md`](./hardcoded-filenames.md) — file finders must verify actual content (column schema), not filename patterns. Same underlying failure mode: testing symbols instead of substance.
- [`forced-sale-file-finder.md`](./forced-sale-file-finder.md) — file finders must verify column contents, not just filename match. Instance of the same class.

## Why this is notable

The field-existence check was written in a registered moment of caution — after the March 30 architecture audit revealed recency was still active in 8 cities despite the ban, the instinct was "block anything that looks like recency." The resulting check was over-broad because it tested the wrong property.

The lesson is sharper than "check weighted behavior, not names." The broader lesson: **validation that catches a regression it wasn't written to catch is doing useful work; validation that false-positives on legitimate state is degrading the operator into a rubber stamp who learns to ignore the gate.** A gate that fails on legitimate state trains the operator to bypass it. A gate that fails only on real violations earns the operator's trust and gets respected.

Referenced in the case study's Layer 4 (Data) as the canonical example of validation that tests the right property at the right level of abstraction.

---

*Solution file schema: date, failure class, symptom, root cause, rule, affected files. Written by Claude Code as part of the Knowledge Compounding Protocol. Read before every new build in the same class.*
