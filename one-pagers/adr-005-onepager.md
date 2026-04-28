# ADR-005: Ban Recency in Distress Scoring (one-pager)

**Decided March 20, 2026; schema-enforced April 1–2, 2026** | Architecture / Scoring

## Context

Through late 2025 and early 2026, seven of the platform's city scoring models included a recency component — `days_since_last_violation`, `days_since_last_payment`, or a normalized variant. The pattern made surface intuitive sense: a fresh signal looks like active distress. It isn't, for the distress definition the product cares about. A new violation means the city is paying attention to that parcel *right now*; a new partial payment means the owner is still trying. Recency biases the score toward parcels that are *visible* rather than parcels that are *entrenched*. Field validation in February–March 2026 surfaced specific cases where recency-weighted models ranked parcels with a single fresh violation above parcels with five years of unpaid taxes — directionally wrong for an experienced operator.

## Decision

Ban recency as a scoring component in any distress model. No `days_since_*`, `weeks_since_*`, or `recency_*` fields contribute weight to a composite distress score. Duration fields (`years_delinquent`, `years_since_first_violation`) are permitted and preferred. Recency-named fields may exist as **metadata** — surfaced in property cards, sort orders, filter inputs — but cannot enter the weighted composite. Enforcement is schema-level: the universal scorer (ADR-003) validates each city's YAML config before deploy and blocks recency-like patterns from contributing weight.

## Alternatives (one line each)

Keep recency, weight it small — a small weight still pulls toward visibility-biased parcels in close cases. Time-decay function with configurable half-life — added a configuration knob per city without a measured improvement to point at. Full ban with manual override flag — once one city has an override, structural enforcement collapses into case-by-case judgment. Drop recency, change nothing else — incomplete; a March 30 audit found two cities where the rule had been "documented complete" while the scored CSVs in production still had recency active.

## Honest limit

The argument against recency is **structural, not measured**. There is no controlled comparison showing recency-weighted distress models produce worse operator-relevant rankings than recency-banned ones on identical input. The case rests on field examples (specific parcels where the recency-weighted score was upside-down versus operator intuition) and the conceptual argument that visibility ≠ distress. Reasonable for a PM decision in an early-stage product; not proof. Duration isn't a clean substitute everywhere — two cities have thin tax-history records, and their composite ceilings are lower than peer cities as a result.

## Revisit triggers

Three or more design-partner operators independently flag that a banned-recency model is *missing* parcels they would expect ranked high, and the missing parcels share a recency-shaped pattern, within 90 days. Schema-gate false positives — validator blocks a non-distress recency-named field more than twice in three months (refine the pattern; don't relax the rule). Synthesis-layer evidence: a controlled comparison of recency-weighted vs. recency-banned synthesis at the agent layer produces a measurable quality delta — codify as a separate ADR.

---

**Full ADR** at [`architecture/adr-005-recency-ban.md`](../architecture/adr-005-recency-ban.md). Includes Chicago's 54.6%-parcel-movement rebuild data, the NYC `score_recency` false-positive incident, and the partial-enforcement gap that drove schema-level enforcement.
