# ADR-003: Universal Scorer with Per-City YAML Configs (one-pager)

**Decided November 2025; schema-hardened April 1–2, 2026** | Architecture / Scoring

## Context

Thirteen cities, each with a structurally different distress scoring model. Signals are similar — tax delinquency, code enforcement violations, sometimes liens — but available fields and useful weights vary because the underlying public data sources don't share a schema. Fort Lauderdale runs a 5-component model; Chicago runs 3 because its CE feed is thinner. The natural-feeling answer is thirteen city-specific scoring scripts — easy to write, easy to change one without touching another. I picked the harder answer.

## Decision

One universal scorer (`scripts/score/score_city.py`) that doesn't know which city it's running. Per-city YAML configs (`config/[city].yml`) declare weights, components, normalization, and city-specific modifiers. Schema validation is non-negotiable: weights sum to 1.0 ±0.01, no recency-named composite components (per ADR-005), required fields present, modifier effects declared explicitly. Schema fail blocks the deploy. New cities get a config file, not new code.

## Alternatives (one line each)

Thirteen city-specific scripts — silent drift; a March 30 audit caught four cities with weight values that didn't match documented weights, only because a compliance validator ran across all configs. Shared library + per-city scripts — partial DRY, leaves city code as a maintenance surface and blurs the config boundary. ML-driven adaptive weights — breaks explainability ("tax balance: 0.45, years delinquent: 0.30" beats "the model learned…") and breaks rebuild-on-config-change auditability.

## Honest limit

The scorer hasn't been stress-tested on a city with a *materially* different data shape. All 13 are some combination of county tax + city CE. A future city with only utility-shutoff records as the distress signal may need a schema extension or a new scorer — open. Pre-validator-era scoring (before April 2) has a measurement gap: the validator confirms current correctness but can't reconstruct how many drifted values existed before it.

## Revisit triggers

A new city's natural model has fewer than two of the existing component categories. A validator-passing config produces field-validated wrong rankings within 30 days of deploy. City configs collectively accumulate more than six explicit modifiers. Institutional buyers become primary revenue and run their own models on the scored outputs — re-litigate the ML-rejection reasoning.

---

**Full ADR** at [`architecture/adr-003-universal-yaml-scorer.md`](../architecture/adr-003-universal-yaml-scorer.md). Includes the four-drifted-weights audit, the modifier-proliferation risk, and the documentation-vs-code mismatch incident that produced a related standing rule.
