# CC Spec Standards — Acceptance Tests & Health Checks

A shared reference that defines what every build spec destined for Claude Code (CC) must include before it ships. Applies across every agent in the fleet that produces CC-bound output.

## Where this fits in the case study

This document is the concrete form of one of the four structural changes shipped in the first week of April 2026, documented in Layer 2 (Orchestration) of the case study:

> *acceptance test standards requiring testable assertions written before any build code, and a Codex external review gate on architecture-touching changes.*

The March 30 architecture audit revealed eight defects across thirteen live city models — scoring components still active after the recency ban, weight drift from documented values, architecture labels that didn't match actual behavior, hardcoded filenames that silently broke enrichment. None of those defects were hypothetical. All shipped to production under the prior standard, which was "describe what to build." The new standard is "describe what correct looks like, in testable terms, before a single line of build code exists."

The case study's pre-April/post-April first-pass acceptance rate (31% → 86%) is the measurement this standard is measured against. The honest limitation noted in the case study applies here: this was one of four co-occurring structural changes, so the rate gain can't be causally assigned to acceptance tests alone. The claim is that structural tightening and rate improvement move together.

## Adaptation notes

This is the DeltaScanner version. The specific assertions reference DS's domain (scored CSVs, parcel data, property enrichment, YAML-configured scoring, 13-city pipeline, web CSV outputs). If you're adapting this standard:

- Keep the structural requirement — every CC-bound spec must include acceptance tests written before build code
- Replace the domain-specific assertions (scoring compliance, enrichment integrity) with whatever "correct" looks like in your system
- Keep the hard-assertion language rule — it's a property of current frontier model behavior, not DS-specific

---

## Why This Exists (original rationale)

Every major incident in DS history traces to the same gap: we defined what to build, but not what "correct" looks like in testable terms. Recency still active after the ban, weight drift in 4 cities, hardcoded filenames breaking enrichment, standing rules pointing to nonexistent files — all would have been caught by a 5-minute test written before the build.

This document defines the standard that applies to **every CC-bound spec**, regardless of which agent or chat produces it.

---

## 1. Acceptance Tests Block — Required in Every CC Spec

### What It Is

A section at the end of every CC spec that lists 5-10 assertions defining "done." CC writes these as automated tests BEFORE writing the build code. The tests become permanent guardrails that run on every future change.

### Template

Add this block at the end of every CC spec, after the build instructions but before any post-ship notes:

```
## Acceptance Tests

CC: Write these as automated tests BEFORE building. All must pass for the build to be considered complete.

### Data Integrity
- [ ] Scored CSV has >0 rows
- [ ] All scores are 0-100 (no negatives, no >100)
- [ ] Zip codes are 5-digit strings (not floats, not "60611.0")
- [ ] No NaN or null values in score column

### Scoring Compliance
- [ ] Weights in YAML config sum to 1.0 (±0.01)
- [ ] No recency-type components in config
- [ ] Architecture type declared (Intersection / Tax-Primary)

### Enrichment
- [ ] Enrichment output column count ≥ previous version
- [ ] Owner name column is >20% populated (or flagged if not)
- [ ] Property type column exists

### City-Specific
- [ ] [Insert city-specific assertions here]
- [ ] [e.g., "Join key match rate >5%"]
- [ ] [e.g., "Folio format matches 13-digit after normalization"]
```

### How to Customize

Every spec should keep the standard assertions (Data Integrity + Scoring Compliance) and add city-specific or feature-specific ones. Examples by spec type:

**New city build:**
- Join key match rate >X%
- Folio/PIN/BBL format matches expected pattern after normalization
- Active status values whitelist matches source documentation
- Scored CSV column count matches spec

**Scoring change:**
- Delta report: no city has >80% of parcels moving >20 points (unless expected and documented)
- Previous enrichment columns preserved
- Web CSV row count within 10% of previous (or deviation explained)

**UI / feature build:**
- Page renders without console errors
- Filter reduces row count vs. unfiltered
- Mobile layout triggers below 768px
- All columns render non-empty for at least 1 row

**Pipeline / infrastructure:**
- Script exits 0 on valid input
- Script exits non-zero on invalid input (missing file, bad format)
- Output file exists at expected path after run
- Validation report generated

### Rules for CC

1. Write the test file FIRST, before any build code
2. Tests must be runnable independently (`python -m pytest tests/` or equivalent)
3. Tests must be committed to the repo alongside the build
4. All tests must pass in the deploy report
5. If a test fails and the failure is expected (e.g., delta report shows large movement due to intentional weight change), document why in the deploy report — don't delete the test
6. **Hard-assertion language only.** Don't use soft filters like "flag if important," "report if concerning," "notice if unusual." Current frontier models (including the one CC runs on) interpret these filters literally and recall drops — even when bug-finding quality improves, reports get suppressed downstream. Write assertions as hard instructions: *report every issue including uncertain ones; tag confidence (high/med/low) and severity (error/warn/info); apply filters downstream in the review step, not in the spec instruction.*

---

## 2. Retroactive Health Check — All Live Cities

### What It Is

A single script that runs every assertion we wish we'd had from the start, across all live cities. Runs as part of the atomic pipeline on every deploy.

### CC Spec: `cc-spec-health-check.md`

```
## Build: Retroactive Health Check Suite

### What to Build

A health check script (`scripts/tests/health_check.py`) that validates all live cities against a standard set of assertions. Runs automatically as the first step of `run_city.py` (atomic pipeline). Also runnable standalone: `python scripts/tests/health_check.py [--city chicago]`.

### Assertions Per City

For EVERY city in `config/*.yml`:

**Scoring**
1. Scored CSV exists at expected path
2. Scored CSV has >0 rows
3. All scores are 0-100
4. No NaN in score column
5. Weights in YAML config sum to 1.0 (±0.01)
6. No recency-type components in config
7. Architecture type is declared (Intersection or Tax-Primary)

**Data Quality**
8. Zip codes are 5-digit strings (no floats)
9. No "undefined" or "NaN" string values in any column
10. Score column is numeric type
11. At least 1 row has a non-empty owner_name (if owner enrichment applied)

**Enrichment Integrity**
12. Enrichment output column count ≥ minimum (6 standard columns)
13. property_type column exists and is >20% populated
14. Every column with <5% population is logged as WARN with exact % populated, column name, and city — report every occurrence, tag severity, do not block pipeline

**Web CSV Consistency**
15. Web CSV in `web/public/data/[city].csv` exists
16. Web CSV row count matches scored CSV row count (post-filter)
17. Web CSV columns match city-config display columns

**Config Consistency**
18. City appears in YAML config
19. City appears in web config (for UI rendering)
20. MSA field populated

### Output Format

```
HEALTH CHECK — [Date]
============================================================
austin        20/20 PASS
nyc           20/20 PASS
philadelphia  20/20 PASS
chicago       19/20 WARN — owner_name <20% populated
baltimore     20/20 PASS
miami         20/20 PASS
...
============================================================
SUMMARY: 13 cities, 258/260 checks passed, 2 warnings
```

### Integration

- Runs as Step 0 of `run_city.py` — before any rescore/enrich/deploy
- If any FAIL (not WARN): pipeline halts, report surfaces to Jeff
- WARNs are logged but don't block
- Standalone mode: `python scripts/tests/health_check.py` (all cities) or `--city chicago` (single city)

### Acceptance Tests (for this build itself)

- [ ] Script runs against all live cities without error
- [ ] Script produces pass/warn/fail per assertion per city
- [ ] Script exits 0 when all pass, non-zero when any fail
- [ ] Script is callable from `run_city.py` as Step 0
- [ ] Script is runnable standalone with `--city` flag
- [ ] At least 1 known issue triggers a WARN (not all green on first run — proves detection works)
```

---

## 3. Global Implementation — Which Agents Need What

### The Principle

Don't duplicate the acceptance test logic in every agent skill file. Instead:

1. This document (`cc-spec-standards.md`) is the shared reference
2. Each agent that produces CC-bound output gets ONE new standing rule pointing here
3. The standing rule is the same everywhere — only the agent's existing output format changes

### Agent Updates Required

| Agent | What It Produces for CC | Change Needed |
|---|---|---|
| **Product Agent** | Feature specs (most CC specs come from here) | Add standing rule + Acceptance Tests block to spec template |
| **QA Triage Agent** | Lane 1 CC reproduction steps | Add standing rule — reproduction steps should include "expected result" assertions |
| **PRR Tracker Agent** | Routes to "write CC build spec" on data intake | Add standing rule — when generating build spec routing, note that spec must include acceptance tests |
| **Data Discovery Agent** | Surfaces data candidates, not CC specs | No change — doesn't produce CC specs |
| **Outreach Drafter Agent** | DM drafts, not CC specs | No change |
| **Deliverable QA Agent** | Pre-flight checklists, not CC specs | No change |

### Standing Rule to Add (identical text for Product, QA Triage, PRR Tracker)

```
**Acceptance Tests Required:** Every spec or output destined for CC must include
an Acceptance Tests section defining 5-10 testable assertions. CC writes these as
automated tests BEFORE building. Standard assertions (scoring compliance, data
integrity, enrichment) come from `cc-spec-standards.md`. City-specific or
feature-specific assertions are added per spec. If this agent produces a CC spec
that omits acceptance tests, the spec is incomplete.

**Hard-Assertion Language:** No soft filters in CC-bound specs — phrases like
"flag if important," "report if concerning," "notice if unusual" cause current
frontier models to filter literally and drop recall. All assertions and
instructions must read as hard instructions: "report every X, tag confidence +
severity, filter downstream." Before sending a spec to CC, scan for soft verbs
(flag, notice, consider, report if) and replace with hard equivalents. See
`cc-spec-standards.md` Section 1 Rule 6 for the canonical pattern.
```

### Product Agent — Specific Addition

The Product Agent's Feature Spec template (the one with Blocks 1-6) needs the Acceptance Tests block inserted after Block 6 (Post-Deploy QA) and before the Codex External Review section. This is where it naturally fits — Block 6 checks the display layer, acceptance tests check the data/logic layer, Codex checks the architecture layer.

### QA Triage Agent — Specific Addition

Lane 1 output (CC reproduction steps) currently has: Steps to Reproduce, Expected Behavior, Actual Behavior. Add:

```
### Verification Assertions
- [ ] [What CC should test to confirm the fix works]
- [ ] [What CC should test to confirm no regression]
```

### PRR Tracker Agent — Specific Addition

In the data intake protocol (Step 6 — "Route to next step"), when the routing says "Write CC build spec," add:

```
Note: CC build spec must include Acceptance Tests per cc-spec-standards.md.
At minimum: scored CSV >0 rows, weights sum to 1.0, no recency,
join key match rate >5%, zip codes are strings.
```

---

## 4. What the Operator Sees

None of this changes the human operator's workflow. The operator still describes what they want in plain English. The agents add acceptance tests to specs automatically. CC writes the tests before the code. The operator sees pass/fail in deploy reports.

The only new thing the operator might see: the health check output in pipeline results showing N cities × 20 checks. If everything's green, it's one line in the report. If something's yellow, it surfaces for a decision.

---

*CC Spec Standards | Adapted from DeltaScanner's production fleet*
*Standing rules: no CC spec ships without acceptance tests; no CC spec ships with soft-filter language.*
