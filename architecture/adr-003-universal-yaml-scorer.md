# ADR-003: Scoring Architecture — Per-City in Production, Universal Scorer Built but Not Shipped

**Status:** Accepted · **Corrected [Jeff sets date]** — an earlier version described the universal scorer as the production architecture. It is built but never shipped; production runs the per-city scripts. Corrected against the live web repo and both pipeline repos.

## Context
Scoring weights had drifted across cities — four ran drifted weights, and Miami's Tier B kept a banned recency signal. With weights hardcoded per script, a change in one place didn't propagate, and drift stayed invisible until it shipped.

## Decision (what's in production)
Each city is scored by its own script (`*_score.py`), dispatched through `run_city.py` via the `CITY_SCRIPTS` table. Scoring weights are centralized in one source of truth — `web/src/config/city-config.json`, read through `shared/scoring_config.py`, which validates weights sum to 1.0 and that no banned recency component is present. 13 of 14 cities load from the shared config (Charlotte still hardcodes). Outputs are normalized by `normalize_csv.py` and committed as static CSVs to the web repo, which Vercel serves with no deploy-time scoring.

## The universal-scorer rebuild (built, not shipped)
A separate consolidation — `deltascanner-pipelines`: one universal config-driven scorer (`score_city.py` + per-city `cities.yml` + `schema.py`) — was built in March 2026 to replace the per-city scripts with a single data-driven scorer. It works and matches its own spec. It has not shipped. That repo has been dormant since Mar 23, has never committed a scored output, and has no deploy path; the production CSVs (committed Apr 14) come from the per-city scripts. The rebuild is banked, not deployed — it earns the cutover when it's validated against the per-city baselines, not before.

## Honest limits
- The universal scorer is real and complete, but unshipped. Production is per-city until the cutover is validated.
- Charlotte is unmigrated from hardcoded weights — the shared config governs 13/14.
- `shared/scoring_config.py` is untracked in git while tracked city scripts import it — a fragility to close.

## Correction note
An earlier version described the universal scorer as the production architecture, citing files from the rebuild repo as if live. Production has always run the per-city scripts. The overclaim — a built-but-unshipped rebuild documented as deployed — was caught by verifying the live web repo's CSVs (committed Apr 14, after the rebuild went dormant Mar 23) against both pipelines.
