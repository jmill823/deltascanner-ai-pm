# Gate Metrics

This page is the forward-logging template for measuring how well the spec → build → deploy gate works in practice. Forward logging began April 22, 2026 — meaning every build cycle from that date forward is recorded against this template, and the rolling history is what gets evaluated when the gate is reviewed.

The case study (`/model-comparison-case-study.md`) makes claims about first-pass acceptance rate and 36-hour defect patterns. Those claims need to remain falsifiable. This page is where the underlying data lives.

---

## What gets measured

Three things, in this order of importance:

1. **First-pass acceptance rate per build cycle.** Of N builds attempted in a cycle, how many passed Block 6 (mechanical post-deploy QA) and Block 7 (acceptance tests) on the first deploy attempt. A build that needed a fix-and-redeploy counts as a fail for the cycle, even if the redeploy passed.
2. **Reject reason categorization.** When a build fails first-pass, what was the cause? Categorized into a fixed scheme (below) so trend analysis is possible across cycles.
3. **30-day defect traceback.** When a defect surfaces in production within 30 days of deploy, trace it back to the original spec to identify which gate would have caught it. This catches gate gaps the first-pass rate misses (a build can pass first-pass and still ship a latent defect).

Forward logging matters because the gate's value isn't its first-pass rate alone — it's the gate's ability to surface its own failure modes. A 100% first-pass rate with rising 30-day defect rates means the gate is too lenient. A 60% first-pass rate with zero 30-day defects means the gate is catching things at deploy that would otherwise reach production.

---

## Table 1 — First-pass acceptance rate

Logged per build cycle. A build cycle is typically a week, but can be a feature-scoped sprint when builds cluster.

| Cycle | Builds attempted | First-pass passes | First-pass rate | Notes |
|---|---|---|---|---|
| 2026-W17 (sample) | 4 | 3 | 75% | One reject; see Table 2 row 2026-W17-R1 |
| | | | | |

**How to log a row.**

- *Cycle*: ISO week (`2026-W17`) or feature scope (`PROD40-property-subtypes`).
- *Builds attempted*: count of distinct CC build sessions that produced a deploy attempt during the cycle. Aborted sessions before deploy do not count.
- *First-pass passes*: count of builds where Block 6 + Block 7 passed on the first deploy attempt. A build that needed a hotfix-redeploy within the same cycle counts as a fail here.
- *First-pass rate*: passes ÷ attempted, rounded to nearest percent.
- *Notes*: links to Table 2 reject rows or any cycle-level context worth flagging.

---

## Table 2 — Reject reason categorization

Logged per first-pass failure. Six categories cover the patterns observed pre-template (the post-mortems behind these categories are in the build-lessons archive).

### Categories

1. **Spec ambiguity.** Spec language admitted two reasonable interpretations. CC chose one; the founder intended the other. Fix: tighter spec prose, or a clarifying example.
2. **Missing or weak acceptance test.** Block 7 didn't include an assertion that would have caught the defect. Fix: extend the standard acceptance test set; update `cc-spec-standards.md`.
3. **Documentation drift.** Spec referenced a file path or component that didn't exist (or had been renamed) in the actual repo. Fix: verify-against-repo step before spec finalization.
4. **Scope creep.** CC built outside the approved scope boundary, even though the spec's "What NOT To Do" section was populated. Fix: tighten the boundary statement, add an explicit STOP-and-flag instruction.
5. **Architecture mismatch.** Spec described an architecture that didn't match the actual repo (e.g., described as 13 scripts, actually a universal scorer). Fix: documentation-against-code audit before spec authoring.
6. **Cross-cutting regression.** Change to one component broke another (city A change broke city B; pipeline change broke frontend display). Fix: add the affected workflow to the standard regression check list.

### Log

| Reject ID | Cycle | Build target | Category | One-line root cause | Fix landed in |
|---|---|---|---|---|---|
| 2026-W17-R1 (sample) | 2026-W17 | City rescore — anonymized | 6 (cross-cutting regression) | Schema change broke list-view column rendering on cities without the new column | `cc-spec-standards.md` regression list extended |
| | | | | | |

**How to log a row.**

- *Reject ID*: cycle + sequential number (e.g., `2026-W17-R1`, `2026-W17-R2`).
- *Build target*: the build, anonymized when needed (city names are public; prospect names and customer-specific configurations are not).
- *Category*: numeric (1–6). If multiple categories apply, log the dominant one; mention secondary in root cause.
- *Root cause*: one sentence. The full post-mortem lives in the lessons archive; this row is a pointer.
- *Fix landed in*: which document or template was updated to prevent the same category from repeating. Empty until the fix ships.

---

## Table 3 — 30-day defect traceback

Logged per production-surfacing defect within 30 days of deploy. This catches what first-pass missed.

| Defect ID | Build (deploy date) | Surfaced (date) | Days post-deploy | Original gate that should have caught it | Why it was missed | Fix |
|---|---|---|---|---|---|---|
| 2026-04-anon (sample) | City build / 2026-04-15 | 2026-04-19 | 4 | Block 7 acceptance test | Test asserted column existence; didn't assert non-empty values | Acceptance test extended to non-empty assertion |
| | | | | | | |

**How to log a row.**

- *Defect ID*: date + short tag (`2026-04-zip-validation`).
- *Build*: which deploy this traces back to, with deploy date.
- *Surfaced*: when the defect was first reported, observed in QA, or detected by an automated check.
- *Days post-deploy*: the gap. Useful for evaluating whether 30 is the right window — if defects keep surfacing on day 35, the window needs to widen.
- *Original gate that should have caught it*: spec gate, Block 6, Block 7, scope-boundary gate, regression check, schema validation. Be specific about which gate is being held accountable.
- *Why it was missed*: one sentence. This is the input for the fix.
- *Fix*: what changed to prevent this category from repeating. Often the fix is the same as a Table 2 row's "Fix landed in" — when that's true, link the rows.

---

## How this page is reviewed

The data is reviewed monthly during the first Sunday setup of each month, at the same review pass that handles the contrarian report and competitive refresh.

The questions for review are stable across months:

- Is first-pass rate trending up or down? If down for 2+ consecutive months, what changed about the spec process?
- Which reject category dominates the cycle? Sustained dominance suggests the corresponding gate is the weakest.
- Are 30-day defects declining as Table 2 fixes ship? If a category keeps showing up in Table 3 despite Table 2 fixes, the fix isn't actually closing the gap.
- Are any categories empty across multiple cycles? Either the gate is working perfectly (good) or the category is misdefined (revisit).

The point of forward logging isn't to produce a vanity metric. It's to keep the gate falsifiable. The case study claims the gate works; this page is where that claim either holds up or doesn't.

---

## Rows are illustrative, not real

The sample rows above are anonymized illustrations of the format, not actual logged events. The live log lives in a separate working file; this page is the template specification.
