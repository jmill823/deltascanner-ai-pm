# Solution — Hardcoded Filenames in Enrichment Scripts

**Date:** April 2, 2026
**Originating build:** Miami enrichment refresh
**Failure class:** Silent data regression from stale file reference
**Status:** Active — rule applied in all subsequent builds

---

## Symptom

Miami's enrichment pipeline produced an output CSV with every row populated, every column present, no errors logged. Post-deploy QA (Block 6) passed every mechanical check. The web CSV loaded cleanly, the UI rendered without issues.

The enrichment output was silently stale. The script had run successfully against a scored CSV from a prior rescore, not the one produced by the current run. No downstream check detected this because the filenames matched the expected pattern and the row count was non-zero.

Caught during ad-hoc review when a scored-CSV value that had been updated in the current rescore did not propagate to the enriched output.

## Root cause

`enrich_parcels.py` referenced the scored CSV by a dated filename string (e.g., `miami_scored_2026-03-27.csv`) rather than dynamically locating the most recent scored CSV. When the rescore step produced a new file with a new date suffix, the enrichment script continued reading the older file because its reference was hardcoded.

The dated filename pattern had been introduced to support rescore versioning — keeping prior scored CSVs on disk rather than overwriting. The enrichment script was not updated at the same time to resolve filenames dynamically.

## Rule

**Enrichment scripts must dynamically locate the current scored CSV. No hardcoded dated filenames in any pipeline step that consumes output from a prior step.**

Implementation pattern:

```python
# REJECTED
scored_csv = pd.read_csv("data/miami_scored_2026-03-27.csv")

# REQUIRED
scored_csv = pd.read_csv(find_latest_scored_csv(city="miami"))
```

Where `find_latest_scored_csv()` resolves the most recent file matching the pattern and — per the related rule in [`forced-sale-file-finder.md`](./forced-sale-file-finder.md) — also verifies that the located file contains the expected columns, not just the expected filename pattern.

## Verification assertions (required in every subsequent spec)

- [ ] No file path string in enrichment or pipeline code matches a dated filename pattern (regex check in pre-commit)
- [ ] Every file read uses a dynamic finder that accepts `(city, file_type)` and returns the latest valid match
- [ ] File finders return the actual modification timestamp of the resolved file in their response, for downstream logging

## Affected files

- `scripts/pipeline/enrich_parcels.py` — primary site of the regression
- `scripts/pipeline/validators.py` — added dated-filename regex check
- `scripts/utils/file_finders.py` — centralized dynamic file resolution (created in response)
- `docs/solutions/hardcoded-filenames.md` — this file

## Related patterns

- [`forced-sale-file-finder.md`](./forced-sale-file-finder.md) — file finders must also verify column contents, not just filename match. The two rules compose: dynamic filename resolution + column-content verification.
- [`pipeline-gate-false-positive.md`](./pipeline-gate-false-positive.md) — validation must check real system state, not surface-level symbols.

## Downstream prevention record

This rule has prevented the same failure class in three subsequent builds (April 6, April 11, April 18). In each case, the pre-commit regex check surfaced a hardcoded-filename pattern during build, and the build was rejected before deploy.

---

*Solution file schema: date, failure class, symptom, root cause, rule, affected files. Written by Claude Code as part of the Knowledge Compounding Protocol. Read before every new build in the same class.*
