# ADR-004: Per-Agent Governance Tiers with Batched Review (one-pager)

**Decided March 2026** | Architecture / Agent Fleet

## Context

Solo founder operating a 10-agent fleet. Without an explicit review architecture, founder review burden becomes the bounding constraint on the system. Two failure modes: approve every action (founder is a manual router; the fleet doesn't compound); approve nothing (drift is invisible until a customer surfaces a problem). A representative city build has 14 distinct steps, 2 requiring human judgment. The other 12 are mechanical, sequential, well-defined enough for an agent — but the founder was operating, not curating.

## Decision

Tier each agent, not each decision. **Gate tier** drafts, surfaces, and waits — used for outreach DMs to named prospects, scoring architecture changes, customer-facing communication. **Reviewer tier** acts, then surfaces what it did at a defined cadence — used for PRR follow-ups to known coordinators, intel-queue insight extraction, build execution within an approved spec. Tier assignment is fixed per agent and based on three criteria: reversibility, downstream blast radius, customer-facing visibility. Cadence: real-time for gate approvals, Sunday setup (45 min) for deep review, Tuesday/Thursday check-ins (15 min) for latency-sensitive batches. Silence in the digest is the failure signal.

## Alternatives (one line each)

Approval-per-action across every agent — bottleneck on founder availability; 14 review touchpoints × 13 cities is 182 review actions for one quarter's pipeline alone. Daily batched review — still two more context switches than the chosen cadence. Weekly batched across every agent — too coarse for PRR statutory deadlines and outreach-DM freshness. Single-tier full autonomy with audit log — action types vary too much in reversibility; a misframed outreach DM is a relationship cost an audit log can't undo. Per-decision risk-tier routing — adds a runtime classification step that itself can fail and forces the founder to learn two layers of system behavior.

## Honest limits

Queue-backup is unmeasured. The system has handled ~10 queued items at once; it hasn't been tested at 30+. Curator-mode collapse is binary — recovery from a reviewer-tier quality issue means dropping into operator-mode for that agent (no graceful partial demotion). Tier assignment heuristics aren't principled — case-by-case judgment, not a formal scoring rubric. Tier-shift triggers ("when does reviewer get demoted to gate?") are uncalibrated.

## Revisit triggers

Gate-tier approval queue exceeds 20 items at any single Sunday setup for 2+ consecutive weeks. Any reviewer-tier agent produces 2+ quality-impacting issues in a single week (auto-demotes to gate, activation review reopens). Reviewer-tier digest goes unread for 3+ weeks. A new agent action type fits neither tier — codify a third tier rather than stretch the binary.

---

**Full ADR** at [`architecture/adr-004-curator-mode-batched-review.md`](../architecture/adr-004-curator-mode-batched-review.md). Includes the 10-agent tier map, the founder-time delta versus pre-March, and the canary-file pattern that surfaces digest breakage.
