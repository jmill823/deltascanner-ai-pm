# Spec Example — Atomic Pipeline Build (PROD30)

A representative build spec following the feature-spec template referenced in the case study and `cc-spec-standards.md`. The PROD30 spec describes an atomic deploy pipeline — all steps pass or nothing ships — with validation gates running before any city rescore-and-deploy.

*Note: This is a reconstructed spec based on the feature-spec template standards and the case study's description of what PROD30 did. The actual PROD30 build is the site of the pipeline-gate false positive documented in [`../solution-files/pipeline-gate-false-positive.md`](./solution-files/pipeline-gate-false-positive.md). The assertions shown here reflect the post-fix state — validation checks the weighted composite, not field existence.*

---

## FEATURE-SPEC-PROD30 | Atomic Deploy Pipeline

**Type:** Pipeline Change
**Secondary:** Infrastructure
**From Intake:** INTAKE-PROD30
**Date:** April 2, 2026
**Estimated Effort:** M (4-6 hours)

---

### What This Does

Replaces the per-city deploy sequence (rescore → enrich → validate → deploy, run manually per city) with a single atomic pipeline script. Every city goes through the same validation gates in the same order. If any gate fails, the pipeline halts and nothing is deployed — no partial state, no half-updated web CSVs, no scoring drift between cities.

### Why We're Building This

The current per-city deploy has two failure modes: (1) an operator running rescore without running the full validation sequence ships undetected scoring drift; (2) a partial failure mid-sequence leaves the system in an inconsistent state (web CSV updated, scored CSV stale, or vice versa). Both have happened. The atomic pipeline enforces the sequence at the script level so neither failure mode is reachable.

---

### CC Build Instructions

#### Scope

Build a single entrypoint `scripts/pipeline/run_city.py` that accepts a city argument and runs the full sequence atomically:

1. **Step 0 — Health Check.** Run `scripts/tests/health_check.py` for the target city. If any FAIL-level assertion triggers, halt. WARN-level issues are logged but don't block.
2. **Step 1 — Rescore.** Run the universal scorer against the city's YAML config. Validate output CSV before continuing.
3. **Step 2 — Enrich.** Run enrichment pipeline. Validate column count and population thresholds.
4. **Step 3 — Validate Config.** Check scoring config compliance: weights sum to 1.0 ± 0.01, no recency-type components in the weighted composite, architecture type declared.
5. **Step 4 — Generate Web CSV.** Filter to display columns, write to `web/public/data/[city].csv`.
6. **Step 5 — Validate Web CSV.** Row count matches scored CSV (post-filter), columns match web config.
7. **Step 6 — Commit and Deploy.** Only reached if all prior steps pass. Single git commit with all generated files.

#### Files Affected

- `scripts/pipeline/run_city.py` — new entrypoint
- `scripts/pipeline/steps/` — new directory, one file per step
- `scripts/pipeline/validation.py` — new module for gate assertions
- `scripts/tests/health_check.py` — existing, called as Step 0
- `config/*.yml` — no changes (pipeline reads, doesn't modify)
- `docs/solutions/` — pipeline writes any new failure-class solution files here (Knowledge Compounding Protocol)

#### Implementation Notes

- The scoring-compliance check at Step 3 must validate the *weighted composite*, not field existence. A config can contain a `score_recency` key as metadata (unweighted, preserved for historical reasons) without violating the recency ban. The assertion is: `sum of weights for any recency-named component in the weighted composite == 0`. See [`../solution-files/pipeline-gate-false-positive.md`](./solution-files/pipeline-gate-false-positive.md) for the failure mode this prevents.
- Every step must emit a structured result object (step name, status, duration, artifacts produced, warnings). The final deploy report is the concatenation of these step results.
- Every file finder in the pipeline must verify column contents, not just filename match. See [`../solution-files/forced-sale-file-finder.md`](./solution-files/forced-sale-file-finder.md) for the pattern.
- Filenames in enrichment must be resolved dynamically from the latest scored CSV, not hardcoded to dated filenames. See [`../solution-files/hardcoded-filenames.md`](./solution-files/hardcoded-filenames.md).

#### What NOT To Do

- **Do NOT run any step out of sequence.** The pipeline is atomic. If a step fails, the next step does not execute. Never add a `--skip-validation` flag. If validation needs to be bypassed for a legitimate reason, that reason is documented in the deploy report and the bypass requires a config change, not a CLI flag.
- **Do NOT allow partial deploys.** Step 6 is the single point at which filesystem changes are committed. No step before it writes to the production web CSV location.
- **Do NOT silently fix config errors.** If a config-compliance check fails, halt and surface the error. Do not auto-correct the config. Do not round weights to force them to sum to 1.0.
- **Do NOT modify scoring weights.** This is a pipeline change only. Scoring model changes are out of scope.

---

### Verification Checklist

- [ ] Running `python scripts/pipeline/run_city.py chicago` executes all steps in order
- [ ] Halting at any failed step produces a deploy report naming the failure
- [ ] No partial filesystem changes are left behind when a step fails mid-pipeline
- [ ] A config with `score_recency` as unweighted metadata passes Step 3 (false-positive regression test)
- [ ] A config with `score_recency` with non-zero weight fails Step 3 with a specific error message
- [ ] Running the pipeline against all 13 cities in sequence produces 13 passing deploy reports (assuming no drift)
- [ ] Deploy report is saved to `reports/pipeline/[city]-[timestamp].md`

---

### Deviation Rules

- Field name mismatch but clearly same data → document in `DEVIATIONS.md`, continue with documented mapping
- Ambiguous implementation choice → take the most conservative interpretation (fail-closed over fail-open), document in deploy report, continue
- Scope expansion opportunity → STOP. Flag to Jeff. Do not build unrequested features.
- Config-compliance ambiguity (e.g., new architecture type not in the allowed list) → halt with a BLOCKING flag in the deploy report. Do not add the architecture type silently. Strategy Chat adjudicates.

---

### Acceptance Tests

CC: Write these as automated tests BEFORE building. All must pass for the build to be considered complete.

#### Pipeline Execution

- [ ] `run_city.py chicago` exits 0 on a clean config and valid data
- [ ] `run_city.py chicago` exits non-zero if Step 0 (health check) fails
- [ ] `run_city.py chicago` exits non-zero if Step 3 (config validation) fails
- [ ] Pipeline halts immediately on first failure — no subsequent steps execute
- [ ] Deploy report exists at `reports/pipeline/[city]-[timestamp].md` regardless of pass/fail

#### Data Integrity

- [ ] Scored CSV has >0 rows after Step 1
- [ ] All scores are 0-100 (no negatives, no >100) after Step 1
- [ ] Zip codes are 5-digit strings in web CSV
- [ ] No NaN or null values in score column of web CSV

#### Scoring Compliance (Step 3)

- [ ] Weights in YAML config sum to 1.0 (±0.01)
- [ ] **Weighted composite contains no recency-named components** (this is the corrected assertion — not field existence)
- [ ] Architecture type is declared (Intersection or Tax-Primary)
- [ ] A config with `score_recency` as unweighted metadata does NOT fail this check (false-positive regression)

#### File Finder Robustness

- [ ] All file finders in the pipeline verify column contents match expected schema, not just filename pattern
- [ ] Enrichment dynamically locates the latest scored CSV — no dated filenames in any enrichment script

#### Web CSV Consistency

- [ ] Web CSV row count matches scored CSV row count (post-filter) after Step 5
- [ ] Web CSV columns match the city's web-config display columns after Step 5

#### Atomicity

- [ ] If Step 4 (web CSV generation) fails, the production web CSV is unchanged from its pre-run state
- [ ] If Step 6 (commit) fails, all prior filesystem changes are rolled back

---

### Codex External Review

This build touches the scoring-compliance validation layer. Architecture-touching changes require external review before deploy.

**Review scope for Codex:**

- Does Step 3's validation correctly distinguish weighted recency components from unweighted metadata?
- Are there any paths through `run_city.py` that could produce a partial deploy (some files updated, others stale)?
- Do the step-by-step validation assertions match the documented behavior in `cc-spec-standards.md` Section 2?
- Is there any state the pipeline depends on that isn't captured in the YAML config or the deploy report (i.e., hidden dependencies that could cause drift between environments)?

Codex review must complete before the first live-city run of this pipeline. If Codex flags issues, route through Strategy Chat before re-deploy.

---

*FEATURE-SPEC-PROD30 | Atomic Deploy Pipeline | April 2, 2026*
*Representative example. Actual DS spec files are internal. The structural template shown here — scope, files affected, implementation notes, what NOT to do, verification checklist, deviation rules, acceptance tests, Codex external review — matches the standard every CC-bound spec must meet per `cc-spec-standards.md`.*
