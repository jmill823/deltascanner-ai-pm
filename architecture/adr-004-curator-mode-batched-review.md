# ADR-004: Per-Agent Governance Tiers with Batched Review

**Status:** Decided March 2026. Currently active across 10-agent fleet.
**Scope:** All product-internal agents (PRR Tracker, CC Intel Queue, Strategy, Product, Outreach Drafter, QA Triage, Data Discovery, Deliverable QA, Claude Code, plus ad-hoc).

---

## Context

The product is operated by a solo founder with a fleet of agents handling everything from public-records-request follow-ups to scoring builds to outreach drafting. Without an explicit review architecture, the founder's review burden is the bounding constraint on the system. Two failure modes:

- **Approve every action.** Founder is a manual router. The agent fleet doesn't compound — it's a thin wrapper over solo work. The system can't grow because the founder is in every loop.
- **Approve nothing, audit nothing.** Agents run unchecked. Quality drift is invisible until a customer or partner surfaces a problem — by which point the failure has already shipped.

The first model was the status quo before March 2026. Counted on a representative city build: 14 distinct steps, 2 of which require human judgment. The other 12 are mechanical, sequential, and well-defined enough that an agent could run them. The founder was operating, not curating.

The goal is not "no founder review." The goal is "founder review concentrated where it actually adds value, on a schedule the founder can sustain."

## Decision

**Tier each agent, not each decision.**

Two tiers:

- **Gate tier.** The agent drafts an action, surfaces it for approval, and waits. No action is sent without explicit approval. Used for: outreach DMs to named prospects, scoring architecture changes, any customer-facing communication.
- **Reviewer tier.** The agent acts, then surfaces what it did at a defined cadence. Audit happens after the fact, in batches. Used for: PRR follow-ups to known coordinators, intel-queue insight extraction, build execution within an approved spec.

Tier assignment per agent is fixed (not per-decision-routed) and based on three criteria:

1. **Reversibility.** A misfired follow-up email to a known coordinator can be retracted with a polite correction; a misdrafted prospect outreach DM can't.
2. **Downstream blast radius.** A scoring architecture change touches every parcel; an acceptance test fix touches one repo.
3. **Customer-facing visibility.** Any action that produces something a buyer or partner sees gets gate tier by default.

Review cadence:

- **Real-time:** crisis events and gate-tier approvals. No queue.
- **Sunday setup (~45 min, weekly):** full QA sweep, PRR status, pipeline status, contrarian/skeptic passes, outreach prep, week-goal setting.
- **Tuesday/Thursday check-ins (~15 min, twice weekly):** DM batch review, pipeline pulse, QA tickets, quick approvals.

For reviewer-tier agents, the audit surface is the agent's wiki section in `/wikis/[agent]/` and the weekly digest. Silence in the digest is the failure signal — a healthy agent produces visible activity, even when the activity is "no PRR responses received this week."

## Alternatives considered

**Approval-per-action across every agent.** Status quo. Rejected — bottleneck on founder availability. Counted concretely: 14 review touchpoints per city build × 13 cities is 182 review actions for one quarter's pipeline work. Multiply by outreach + PRR + intel and the founder has no time left for buyer conversations or product decisions.

**Daily batched review.** Rejected — daily is still too frequent for sustained solo ops. Anything daily is two more context switches than weekly + twice-weekly. Empirically, the highest-value review work happens during Sunday setup; weekday reviews are top-up, not deep-pass.

**Weekly batched review across every agent.** Too coarse. Some action types are time-sensitive: PRR follow-ups have statutory deadlines that don't wait for Sunday, and outreach DM drafts that sit for a week stop being current. Twice-weekly Tuesday/Thursday check-ins handle the latency-sensitive work. Sunday handles the deep work.

**Single-tier: full autonomy with audit log for every agent.** Rejected — action types vary too much in reversibility. An outreach DM to a named prospect that opens with the wrong frame is a relationship cost an audit log can't undo. A misfired PRR auto-acknowledgment is a one-line correction email. Same audit pattern for both is wrong; the tier needs to vary.

**Per-decision risk-tier routing.** Each individual decision gets routed to a tier based on classification at runtime. Rejected for complexity. Adds a runtime classification step (which itself can fail), makes the audit trail harder to follow ("why did this PRR follow-up route to gate when last week's didn't?"), and forces the founder to learn two layers of system behavior — agent identity *and* decision-class. Per-agent tiering is coarser but legible.

**Tier-per-agent with explicit re-evaluation triggers.** Chosen. Tier is fixed at agent activation. Tier shifts (gate → reviewer or vice versa) require an explicit trigger: sustained good performance to relax, sustained quality issues to tighten. Currently no agent has shifted tier; the architecture is young enough that all initial assignments are still in their first evaluation window.

## Consequences

- Ten agents currently mapped. Strategy / Product / Outreach Drafter = gate tier. PRR Tracker / CC / CC Intel Queue / QA Triage / Data Discovery / Deliverable QA = reviewer tier.
- Founder review time has moved from continuous-availability to roughly 3.5 scheduled hours per week (45 min Sunday + 15 min × 2 weekday check-ins + ~1.5 hour ad-hoc gate approvals during the week).
- Buyer-facing time has approximately doubled. Estimated from comparing pre-March vs post-March weekly schedules; not measured precisely.
- The "silence is the signal" pattern works for a reviewer-tier agent only when the agent has a visible wiki/digest output. Two agents (Data Discovery, Deliverable QA) currently have skill files but no live activity — for them, the audit surface doesn't yet exist, and they're effectively neither gate nor reviewer until they're activated.
- Failure mode visibility is concentrated in the weekly digest. If the digest itself is broken, all reviewer-tier agents become invisible simultaneously. The PRR Tracker's `_digest.md` is the canary file — its absence is the system-broken signal.

## Honest limits

**Queue-backup failure mode is unmeasured.** If batched approvals stack up at a gate-tier agent (e.g., a busy outreach week that produces 30 DM drafts), the founder either powers through and reviews superficially (quality erosion) or lets the queue grow (latency erosion). I don't have a measured threshold for when this breaks. Empirically the system has handled ~10 queued items at once; it hasn't been tested at 30+.

**Curator-mode collapse is binary, not graceful.** If a single reviewer-tier agent starts producing quality issues, recovery means dropping into operator-mode for that agent — read every output before it ships, fix the agent skill or prompt, re-promote when stable. There's no "partial demotion" that keeps some autonomy intact. In practice this has happened twice (the PRR Tracker's first week had a false-positive bounce-classification issue, which forced two days of operator-mode); recovery worked but the overhead spike was costly.

**Tier assignment heuristics aren't principled.** "Outreach as gate" is principled — outreach is customer-facing. "Strategy as gate" is principled — strategy chat produces architectural proposals that need founder approval. "PRR Tracker as reviewer" was assigned partly because volume is high and individual reversibility is high — also principled. But there's no formal scoring rubric that says "any agent meeting threshold X gets reviewer tier." Assignments are case-by-case judgment. As the fleet grows, this lack of formalism may produce inconsistent assignments.

**The tier-shift trigger isn't yet codified.** When does a reviewer-tier agent get demoted to gate? The current rule of thumb is "more than one quality-impacting incident per week," but that's not measured systematically and the threshold is uncalibrated.

## Revisit triggers

- **Sustained queue backup.** If a gate-tier agent's approval queue exceeds 20 items at any single Sunday setup for 2+ consecutive weeks, the agent's tier or the cadence needs review.
- **Reviewer-tier quality incidents.** If any reviewer-tier agent produces 2+ quality-impacting issues in a single week (defined: any output that requires the founder to operator-mode-fix and re-issue), that agent demotes to gate immediately, and the activation review for that agent gets reopened.
- **Audit debt at a reviewer-tier agent.** If un-reviewed agent output accumulates beyond 3 weeks for any reviewer-tier agent (digest unread for 3+ weeks), the cadence isn't sustainable — either the audit surface needs simplification or the agent needs less ambient activity.
- **A new agent action type that doesn't fit either tier.** The current binary may not cover every future action class — partner-shared deliverables, for instance, sit somewhere between "customer-facing" (gate) and "internal" (reviewer). When a third tier becomes inevitable, codify it rather than stretching the existing two.

## Related decisions

- **Hybrid wiki architecture (April 16, 2026).** Provides the audit surface that batched review depends on. Without `/wikis/[agent]/` digests, batched review degrades into "ask the agent what happened," which is operator-mode in a different costume.
- **Session close handoff requirement (April 3, 2026).** Each agent session that produces changes ends with a handoff document. Reduces the "what happened?" reconstruction time during review.
- **External review gate (April 2, 2026).** An independent agent (Codex) audits scoring/architecture builds. Curator-mode extends from "founder reviews agents" to "agents review agents," which the founder can audit at lower frequency.
- **ADR-001: Knowledge Compounding Protocol.** Solution files at `docs/solutions/` written after every build. Reduces the founder's need to be in the build loop for context-transfer reasons specifically.
