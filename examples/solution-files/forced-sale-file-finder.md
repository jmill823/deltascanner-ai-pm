# Solution — Forced Sale File Finder Selected Wrong Schema

**Date:** April 4, 2026
**Originating build:** PROD40 — Chicago forced sale integration
**Failure class:** File finder matched on name, not on content schema
**Status:** Active — rule applied in all subsequent builds

---

## Symptom

During the PROD40 Chicago build, the forced-sale integration script selected a file whose filename matched the expected pattern but whose column schema did not contain the expected fields. The script did not raise an error — it attempted to read columns that did not exist, and the affected fields silently became null in downstream output.

This would have shipped as a silent data loss: the forced-sale augmentation would appear to run successfully, the output CSV would contain the expected column count, but the forced-sale-derived fields would be empty for every row.

Claude Code caught the gap autonomously during build execution and wrote this solution file before continuing. The fix was applied, the build resumed with correct behavior.

## Root cause

The file finder `find_latest_forced_sale()` selected files by matching against a filename pattern (prefix `forced_sale_`, CSV extension, most recent modification time). A non-forced-sale file had been inadvertently saved to the forced-sale directory with a compatible filename prefix but different columns. The finder selected it because the filename pattern matched and the modification time was most recent.

The underlying assumption — that filename convention reliably implies schema — was never validated. In a pipeline where multiple sources can write to the same directory, filename conventions are not a sufficient schema guarantee.

## Rule

**File finders must verify column contents, not just filename match. Every file finder returns a validated file or raises.**

Implementation pattern:

```python
# REJECTED — filename match only
def find_latest_forced_sale(city):
    return max(glob(f"data/{city}/forced_sale_*.csv"), key=os.path.getmtime)

# REQUIRED — filename + schema validation
def find_latest_forced_sale(city):
    candidates = sorted(glob(f"data/{city}/forced_sale_*.csv"),
                        key=os.path.getmtime, reverse=True)
    for path in candidates:
        if _schema_matches(path, expected_columns=FORCED_SALE_SCHEMA):
            return path
    raise FileNotFoundError(
        f"No forced-sale file in data/{city}/ matches expected schema: "
        f"{FORCED_SALE_SCHEMA}"
    )
```

Where `_schema_matches()` reads the header row and verifies all required columns are present. The expected schema is defined alongside the finder, not inferred.

## Verification assertions (required in every subsequent spec that uses file finders)

- [ ] Every file finder function verifies column contents before returning a match
- [ ] Every file finder raises on no valid match, rather than returning the most recent file regardless
- [ ] Expected schemas are defined as module-level constants, not inferred from the first file read
- [ ] File finder contract includes: input parameters, expected schema, return type, raise conditions

## Affected files

- `scripts/pipeline/forced_sale_integration.py` — primary site
- `scripts/utils/file_finders.py` — added schema verification to all finder functions
- `scripts/utils/schemas.py` — new module for schema constants (created in response)
- `docs/solutions/forced-sale-file-finder.md` — this file

## Related patterns

- [`hardcoded-filenames.md`](./hardcoded-filenames.md) — file finders must resolve filenames dynamically, not reference dated strings. The two rules compose: dynamic filename resolution + column-content verification.
- [`pipeline-gate-false-positive.md`](./pipeline-gate-false-positive.md) — validation must check real system state, not surface-level symbols. File finders matching on name rather than content is an instance of this broader failure mode.

## Why this is notable

This regression was caught end-to-end by the Knowledge Compounding Protocol working as designed:

- Claude Code encountered a situation where a file finder returned a schema-mismatched file
- Rather than accept the silent null output, CC surfaced the gap, implemented the fix autonomously, and wrote this solution file
- No human in the loop was required
- The rule here now gates every subsequent file-finder build

The protocol working end-to-end on a real regression — not a synthetic test case — is the evidence that structured cross-session memory generalizes beyond the specific class of failure that prompted its creation. This example is referenced in the case study's Layer 4 (Data) as the clearest demonstration of autonomous generalization from a prior lesson.

---

*Solution file schema: date, failure class, symptom, root cause, rule, affected files. Written by Claude Code as part of the Knowledge Compounding Protocol. Read before every new build in the same class.*
