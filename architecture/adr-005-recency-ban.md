# ADR-005: Ban Recency in Distress Scoring

**Status:** Decided March 20, 2026. Schema-enforced April 1–2, 2026.
**Scope:** All city scoring models (currently 13 cities, ~223,000 scored parcels).

---

## Context

The product scores residential parcels for distress by combining two public signals: tax delinquency and code enforcement (CE) violations. Through late 2025 and early 2026, seven city scoring models included a *recency* component — `days_since_last_violation`, `days_since_last_payment`, or a normalized variant. On March 20, 2026 I removed all of them, replaced what could be replaced with duration components, and added schema-level enforcement two weeks later to prevent recency from coming back through the back door.

This ADR captures why.

The recency pattern made surface intuitive sense. A fresh signal looks like active distress: recently filed violation, recently missed payment, recently elevated case stage. That feels worse than a stale one.

It isn't. Or at least it isn't for the distress definition the product cares about.

The composite score is meant to capture **operator-relevant distress** — parcels where the owner is structurally over-leveraged, behind on obligations long enough that the situation is unlikely to self-correct, and at meaningful risk of forced disposition. Recent activity is partly the opposite signal. A new violation means the city's enforcement system is paying attention to that parcel *right now*. A new partial payment means the owner is still trying. Both can show up on genuinely distressed parcels, but the recency signal weighted positively biases the score toward parcels that are *visible* rather than parcels that are *entrenched*. What carries the operator-relevant signal is **duration** — years delinquent, time elapsed since first violation, accumulated stage progression — not days since the most recent event.

Field validation in February and March 2026 surfaced specific cases where recency-weighted models ranked parcels with a single fresh violation above parcels with five years of unpaid taxes and no recent activity. That's the opposite of what an experienced operator would prioritize. Recency wasn't just imprecise — it was directionally wrong for a meaningful subset of parcels.

## Decision

**Ban recency as a scoring component in any distress model.** Replace with duration where the underlying field supports it.

Specifically:

- No `days_since_*`, `weeks_since_*`, or `recency_*` fields contribute weight to a composite distress score.
- Duration fields (`years_delinquent`, `years_since_first_violation`) are permitted and preferred.
- Recency-named fields may exist as **metadata** — surfaced in property cards, sort orders, filter inputs — but cannot enter the weighted composite.

Enforcement is schema-level. The universal scorer validates each city's YAML config before deploy and blocks recency-like patterns from contributing weight. Policy without tooling-level enforcement was insufficient (see Consequences).

## Alternatives considered

**Keep recency, weight it small.** Status quo before March 20. Rejected — a small weight still pulls the score toward visibility-biased parcels in close cases. Reducing the weight reduced the magnitude of the wrong direction without fixing the direction.

**Time-decay function with configurable half-life.** The framing: maybe recency is wrong as a positive contribution but right as part of a decay envelope on duration. Rejected — added a configuration knob per city without a measured improvement to point at, and the structural argument against recency-as-positive-signal doesn't cleanly translate into "use it as a decay curve instead." The audit and explanation surface costs weren't worth taking on without evidence.

**Full ban with manual override flag.** Rejected — manual overrides erode the rule. Once one city has an override, the structural enforcement collapses into case-by-case judgment, and the audit trail for "why does this city's score work differently" multiplies. The rule needs to be binding or dropped. "Binding except when not" is the worst version.

**Drop recency, change nothing else.** Rejected as incomplete. The March 20 batch removed recency components from seven models, but a March 30 audit found two cities where the rule had been "documented as complete" while the scored CSVs in production still had recency active — the code change had been committed but the rescore hadn't run. Schema validation was added April 1–2 because policy without tooling-level enforcement degrades silently.

## Consequences

- Seven city models rebuilt in the March 20 batch. One city (Chicago) saw 54.6% of parcels move more than 20 points after rescore — large, but expected: tax-only parcels had been suppressed by absent CE-recency signal. The post-correction ranking matches operator intuition more closely on field validation.
- The universal scorer enforces the ban through schema validation. Recency-named composite weights are blocked at config-load time, before any score is computed. Any future config change that tries to reintroduce recency-as-weight fails the deploy gate.
- One false positive surfaced April 3: NYC had a `score_recency` field as metadata only, with zero composite weight. The initial gate flagged it on field-name match. The refined gate checks the *weighted* composite, not field existence — a recency-named field with zero weight is permitted as metadata. Worth noting because the lesson is general: validation gates should check the operative property (weight) rather than the surface property (name).
- Customer-facing scoring documentation got simpler. The composite is duration-weighted across tax and CE signals. Recency, if it appears at all, is a sort or filter input — not part of the conviction score.

## Honest limits

The argument against recency is **structural, not measured**. There is no controlled comparison showing recency-weighted distress models produce worse operator-relevant rankings than recency-banned ones on identical input data. The case rests on field examples (specific parcels where the recency-weighted score was upside-down versus operator intuition) and on the conceptual argument that visibility ≠ distress. That's a reasonable foundation for a PM decision in an early-stage product. It isn't proof.

The structural argument plausibly extends to **synthesis layers** — agents weighing recent insights more heavily during knowledge synthesis, on the same intuition that fresh = signal. The same critique applies (newest insights haven't accumulated downstream evidence yet). I haven't codified that as a separate rule because the synthesis-layer failure mode hasn't been verified at the scale the scoring failure was. Noted as an extension under the same principle, not as a separate banned pattern.

Duration isn't a clean substitute everywhere. Two cities have thin tax-history records (recently digitized, sparse historical data). For them, removing recency loses proportionally more useful signal than duration recovers. Their composite ceilings are lower than peer cities — the model surfaces less conviction rather than padding scores with a directionally wrong signal. Whether that's the right tradeoff for those two cities is open; it has not produced a customer complaint to date.

## Revisit triggers

- **Operator field feedback.** If three or more design-partner operators independently flag that a banned-recency model is *missing* parcels they would expect ranked high, and the missing parcels share a recency-shaped pattern (one big recent event, no long history), the ban may be over-broad. Threshold: 3 independent flags within a 90-day window.
- **Schema-gate false positives.** If the validator blocks a non-distress recency-named field more than twice in three months because of overly broad pattern matching, the gate is too aggressive. Refine the pattern; don't relax the rule.
- **Synthesis-layer evidence.** If a controlled comparison of recency-weighted versus recency-banned synthesis at the agent layer produces a measurable quality delta in either direction, codify the result as a separate ADR rather than extending this one.

## Related decisions

- **Schema validation requirement** (April 1–2, 2026): all scoring config changes pass through `schema.py` before deploy. Origin partly here — policy without enforcement was insufficient.
- **Rule-change verification** (March 30, 2026): a scoring rule is not "complete" until the live scored CSV reflects it. Origin: the partial-enforcement gap that surfaced this rule's incomplete rollout.
- **External review gate** (April 2, 2026): scoring/architecture/config changes get an external code-audit pass before deploy. Same reinforcement pattern — catch documentation/code mismatches before they ship.
