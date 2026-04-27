# ADR-003: Single Universal Scorer with Per-City YAML Configs

**Status:** Decided November 2025 (initial implementation). Re-verified and schema-hardened April 1–2, 2026.
**Scope:** All distress scoring across the platform's 13 cities.

---

## Context

Each city's distress scoring model is structurally different. The signals are similar — tax delinquency duration, code enforcement violation count and severity, sometimes lien data, sometimes assessment data, sometimes a city-specific modifier — but the *available* fields and the *useful* weights vary city by city because the underlying public data sources don't share a schema.

A clean example: Fort Lauderdale uses a 5-component model (assessed fines, enforcement stage, CE lien flag, tax balance, years delinquent) because the city's CE data is unusually rich. Chicago uses a 3-component model (tax balance, tax years, violation severity) because the CE feed is thinner. Both are correct for their data shape; copying Fort Lauderdale's weights to Chicago would produce nonsense scores.

The natural-feeling first answer to this problem is to write thirteen city-specific scoring scripts. Each script knows its city's fields, hard-codes its weights, normalizes its inputs the way its data needs. Easy to write. Easy to change one city without touching another.

I picked the harder answer.

## Decision

**One universal scorer with per-city YAML configs.**

The scorer (`scripts/score/score_city.py`) doesn't know which city it's running. The config (`config/[city].yml`) declares the weights, the components, the normalization method, and any city-specific modifiers (×1.10 for ≥3-year delinquency, +5pt for multi-violation-type flag). The scorer reads the config, verifies it through a schema validator (`scripts/utils/schema.py`), and produces the scored CSV. New cities don't get new scoring code — they get a new config file.

Schema validation is non-negotiable:

- Weights must sum to 1.0 within ±0.01.
- No recency-named composite components (per ADR-005).
- Required fields must be present.
- Modifiers must declare their effect explicitly (multiplier or additive).

If any check fails, the deploy is blocked.

## Alternatives considered

**Status quo: thirteen city-specific Python scripts.** The most natural model. Rejected — each script duplicates 80% of the others. Drift is silent: a fix in one script doesn't propagate. A March 30, 2026 audit found four cities with weight values that didn't match the documented weights, and the only reason any of them were caught was a compliance validator running across all configs. Without centralization, drift had been accumulating without detection.

**Shared library + per-city scripts.** Partial DRY. The shared library holds normalization, schema enforcement, IO. Each city script imports the library and supplies its own weights. Rejected — still leaves city-specific code as a maintenance surface, and the configuration boundary blurs. "Why is Charlotte's modifier in code instead of config?" is a question that comes up. The principle that *all* city behavior lives in the config (and *only* in the config) becomes mushy.

**Universal scorer with per-city YAML configs.** Chosen.

**ML-driven adaptive weights (per-city or per-segment).** Considered as a future direction. Rejected for now — the product is sold partly on explainability. An operator who asks "why is this parcel ranked 87?" needs an answer they can verify. "The model learned that this city's parcels with this combination of attributes correlate with distress" is a worse answer than "tax balance: 0.45, years delinquent: 0.30, violation severity: 0.25." Adaptive weights also break the rebuild-on-config-change pattern — if weights drift, the model behaves differently between deploys for reasons that aren't human-auditable. Revisit if there's evidence that hand-tuned weights are systematically miscalibrated.

## Consequences

- New city pipelines take 1–2 days of config work versus the ~1 week of script-writing they would otherwise. The bottleneck is data acquisition, not scoring.
- A `schema.py` change propagates to all 13 cities. The April 1–2 schema-hardening pass added the recency ban, weight-sum verification, and modifier explicitness — all 13 configs revalidated automatically.
- Property type code translation, address normalization, and similar data-shape work happens *upstream* of the scorer (in `normalize_csv.py` per city). The scorer stays data-shape-agnostic. A city's quirky source codes never reach scoring logic; they're translated to canonical types at normalization. This rule (don't make the scorer accommodate one city's data shape) was tested when Houston needed HCAD `state_class` codes joined and when Fort Worth needed SPTB code mapping — both done at normalization, scorer untouched.
- The four drifted-weight cities found in the March 30 audit were the empirical case for this architecture's value. Three had weight values that no one had touched in months but had been silently wrong. The schema validator now blocks that failure mode at deploy.
- One April 2 incident: a Codex external review caught that the prior week's documentation described a different architecture (`city-config.json` + 13 scripts) than what was actually deployed (YAML configs + universal scorer). The code was right; the docs were wrong. This produced a rule (standing rules must verify against actual repo paths before being documented) more than it changed the architecture.

## Honest limits

The scorer hasn't been stress-tested on a city with a *materially* different data shape. All 13 cities to date are some combination of county-level tax delinquency + city-level code enforcement. If a future city brings, say, only utility shutoff records as the distress signal, the scorer's component model may need to extend beyond the current weight-sum-to-1.0 pattern. Whether that's a config-schema extension or a new scorer is open.

The four-drifted-weights finding from March 30 was caught by the validator, not by manual review. That's the architecture working as designed — but it also means I don't know how many other drifted values existed in the pre-validator era and went undetected. Some scored CSVs from that period are still in production. They're not actively wrong (validation has since reverified them), but the historical scoring quality before April 2 has a measurement gap.

The ML-rejection reasoning leans hard on explainability. If a future buyer doesn't care about the operator-readable score breakdown — institutional buyers, for instance, who are running their own models against the scored outputs — the explainability case weakens, and adaptive weights become more defensible. The decision is correct for the current ICP but will need re-litigation as the buyer mix shifts.

City-specific modifiers (×1.10 multipliers, +5pt additives) are documented in config but proliferate over time. Three cities currently have one modifier each. If the count grows past, say, six total modifiers across the fleet, the modifier system itself starts to feel like a parallel scoring layer hiding inside the config — which would defeat the legibility argument.

## Revisit triggers

- **A new city's data shape requires component types that don't fit the current schema.** Threshold: any city where the natural model has fewer than two of the existing component categories (tax-balance, tax-duration, violation-count, violation-severity, lien-flag).
- **Weight drift recurs after schema validation.** If a validator-passing config produces field-validated wrong rankings within 30 days of deploy, the weight-tuning process itself is wrong, not just the values.
- **Modifier proliferation.** If the city configs collectively accumulate more than six explicit modifiers, the modifier subsystem warrants its own ADR.
- **Buyer mix shifts toward adaptive.** If institutional buyers become the primary revenue segment and they're running their own models on top of the scored outputs, revisit the ML-rejection reasoning.

## Related decisions

- **ADR-005: Recency ban** — enforced through this architecture's schema validator.
- **External review gate (April 2, 2026)** — backstop for documentation-vs-code drift; this architecture made the gate worth doing because the code was complex enough that documentation drift was plausible.
- **Rule-change verification (March 30, 2026)** — a scoring rule isn't complete until the live scored CSV reflects it. The validator + universal scorer make this verification automatic instead of manual.
