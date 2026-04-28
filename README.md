# DeltaScanner as an AI Product Management Case Study

A model upgrade broke my agent system last week. The fix was a file I'd never thought to write — an Operator Profile encoding how I work, model-agnostic, ported across the fleet. The layer it created didn't appear in any architecture diagram I'd seen.

That's the case study. The architecture wasn't designed; it was discovered through operating.

I'm a non-technical solo founder. I do not write build code. The agents write, build, deploy, and surface work for review. I approve, reject, or defer at scheduled gates. Between gates, my inbox is empty by design. Every layer in the stack exists because something specific broke and I encoded the fix.

This repo is the working artifact of that approach. It contains the full case study, a shorter distillation, the compounded intelligence brief that informs the framing, and the supporting documents that make the case study verifiable rather than rhetorical — real spec standards, real handoff contracts, real solution files showing the system catching real regressions.

---

## What this repo is

A public record of one operator running an eight-agent fleet to ship a distressed property intelligence platform across thirteen U.S. markets. The case study walks five architectural layers, top-down. Each layer has a documented origin failure that produced it:

- **Layer 1 — Interaction.** Operator Profile + product surface. Origin: model upgrade broke calibration.
- **Layer 2 — Orchestration.** Curator-mode batched review with handoff contracts. Origin: three substrate miscalibrations in one session.
- **Layer 3 — Specialized Agents.** Eight agents with single responsibilities and versioned skill files. Origin: handoff failures and lane drift.
- **Layer 4 — Data.** The Knowledge Compounding Protocol — structured cross-session memory the fleet reads before every build and writes to after every build. Origin: cross-session knowledge was 100% dark.
- **Layer 5 — Model API.** Maker-checker separation across four model families. Origin: same-model-as-its-own-auditor failure caught by independent review.

## What this repo is not

A framework. A claim that this approach generalizes without modification. A finished system. DeltaScanner is in operation, not in retrospect. What's published here is what I learned running the system I have now — measured against the specific evidence available, with the limitations named where I can't defend a stronger claim.

The "designed by operating" thesis is a description of the path that produced this system, not a methodology that generalizes to every multi-agent system. Whether it would have been faster to design these five layers upfront is unknowable; what's testable is whether the layers I have hold as the system scales.

---

## Architecture diagram

![DeltaScanner agent fleet — five-layer architecture](./architecture/fleet-diagram.svg)

Visual companion to the case study. Each layer in the diagram corresponds to a section in the [case study one-pager](./one-pagers/case-study-onepager.md) and its long-form source, naming the failure that produced it.

---

## Repository contents

### One-pagers (60-second reads)

All eight one-pagers live in [`/one-pagers/`](./one-pagers/). Each follows the same compressed structure: context, decision or insight, what it reveals, honest limit. Each links to its long-form source for readers who want to drill down.

- [`case-study-onepager.md`](./one-pagers/case-study-onepager.md) — The five-layer architecture, top-down. The case study compressed.
- [`model-comparison-onepager.md`](./one-pagers/model-comparison-onepager.md) — *"The Layer Nobody Designs For."* The 36-hour pattern that surfaced the Operator Profile.
- [`adr-001-onepager.md`](./one-pagers/adr-001-onepager.md) — Knowledge Compounding Protocol. Decision, alternatives, honest limit, revisit trigger.
- [`adr-002-onepager.md`](./one-pagers/adr-002-onepager.md) — Multi-Provider Maker-Checker Routing. Decision, alternatives, honest limit, revisit triggers.
- [`adr-003-onepager.md`](./one-pagers/adr-003-onepager.md) — Universal Scorer with Per-City YAML Configs. Decision, alternatives, honest limit, revisit triggers.
- [`adr-004-onepager.md`](./one-pagers/adr-004-onepager.md) — Per-Agent Governance Tiers with Batched Review. Decision, alternatives, honest limits, revisit triggers.
- [`adr-005-onepager.md`](./one-pagers/adr-005-onepager.md) — Ban Recency in Distress Scoring. Decision, alternatives, honest limit, revisit triggers.
- [`operator-profile-onepager.md`](./one-pagers/operator-profile-onepager.md) — The Layer 1 calibration file. Why it exists, what the structure encodes, generalization limit.

### Long-form sources

The long-form case study (~3,500 words) and the model-comparison sister artifact are available on request. The one-pagers in `/one-pagers/` carry the canonical compressed versions. The long-form files walk each layer in depth: what it does, the failure that created it, the evidence that it's working, and consolidated honest limitations.

### Compounded intelligence

- [`briefs/ai-pm-brief.md`](./briefs/ai-pm-brief.md) — Six principles compounded across thirteen batches of intelligence extraction. Live cross-references to public sources (Cat Wu, Mahesh Yadav, others) that converged independently on the operating model the case study describes. Auto-regenerated each compile cycle from the source intel repo.

### Supporting artifacts

- [`handoff-template.md`](./handoff-template.md) — The structured format every agent in the fleet uses to close a session. Concrete form of the "every handoff follows a contract" claim in Layer 2.
- [`examples/cc-spec-standards.md`](./examples/cc-spec-standards.md) — The shared standard for every build spec destined for Claude Code. Acceptance tests required, written before build code. One of the four structural changes that moved first-pass build acceptance from 31% to 86%.
- [`examples/spec-prod30-atomic-pipeline.md`](./examples/spec-prod30-atomic-pipeline.md) — Representative build spec following the template. The actual PROD30 build is the site of a pipeline-gate false positive, documented in the solution files.
- [`examples/solution-files/`](./examples/solution-files/) — Three structured solution files from the Knowledge Compounding Protocol. Each one encodes a specific failure class the system caught in April 2026, with the rule that prevents recurrence. These are the files the fleet reads before every build.
- [`examples/operator-profile.md`](./examples/operator-profile.md) — Sanitized v5 of the Layer 1 calibration file. Model-agnostic behavioral contract that ports across model versions. The v1→v5 changelog records five discipline sections added in real time as failure modes surfaced in operating sessions across April 25–27, 2026.
- [`agent-fleet.md`](./agent-fleet.md) — Layer 3 expansion. Per-agent breakdown of the nine-agent fleet plus the CC spec standards doc: scope, what-it-does-NOT-do, handoff format, escalation paths, and version history for each.
- [`metrics/gate-metrics.md`](./metrics/gate-metrics.md) — Forward-logging template for first-pass acceptance rate, six-category reject-reason scheme, and 30-day defect traceback. Sample rows are illustrative; the live log lives in a separate working file. Exists to keep the case-study claim about gate effectiveness falsifiable.

### Architecture Decision Records

- [`architecture/adr-001-knowledge-compounding-protocol.md`](./architecture/adr-001-knowledge-compounding-protocol.md) — The decision to write structured solution files after every build and read them before every future build. Five alternatives considered. Honest limits names the causal-isolation problem, retrieval-degradation threshold, generalization risk, and Goodhart enforcement gap.
- [`architecture/adr-002-model-routing.md`](./architecture/adr-002-model-routing.md) — The decision to split the agent fleet into three model tiers (Opus / Sonnet / GPT-4o-mini) with Codex as external checker. Five alternatives considered, including dropping skeptic review entirely. Honest limits names the borrowed-principle risk, unmeasured cross-family error correlation, unvalidated cost claim, and Sonnet-tier structural prior.
- [`architecture/adr-003-universal-yaml-scorer.md`](./architecture/adr-003-universal-yaml-scorer.md) — The architecture choice of one universal scorer plus 13 per-city YAML configs, versus 13 city-specific scripts. Includes the four-drifted-weights audit finding from March 30 as the empirical case for centralization, and the explainability argument against ML-driven adaptive weights.
- [`architecture/adr-004-curator-mode-batched-review.md`](./architecture/adr-004-curator-mode-batched-review.md) — The choice to tier each agent (gate vs reviewer) rather than per-decision routing, and the batched-review cadence (Sunday + Tue/Thu) the tier architecture supports. Includes four honest-limits sections: queue-backup unmeasured, curator-collapse binary, tier-assignment heuristic, tier-shift trigger uncodified.
- [`architecture/adr-005-recency-ban.md`](./architecture/adr-005-recency-ban.md) — The March 20, 2026 decision to ban recency components in distress scoring, replaced with duration where applicable. Schema-enforced April 1–2. Includes alternatives, honest limits on the structural-not-measured argument, and revisit triggers.

---

## How to read this repo

If you have 60 seconds, [`/one-pagers/`](./one-pagers/) is the front door. Each one-pager is a compressed entry point with a link to its long-form source.

If you read the full case study, the supporting artifacts are the verification layer. Every mechanism the case study names has a corresponding file here.

If you came in through a supporting artifact (someone linked you a spec standard or a solution file), start with [`one-pagers/case-study-onepager.md`](./one-pagers/case-study-onepager.md) to get the argument the artifacts support.

---

## About DeltaScanner

DeltaScanner is a distressed property intelligence platform — live across thirteen U.S. markets, ~223,000 scored parcels. The product thesis is workflow compression: turn validated parcel-level distress signal into operator decisions in seconds. The AI-PM case study is what I learned running it.

The case study is an artifact of work in motion. It's not a retrospective written at the end of a completed project. It's a snapshot of a system that's still operating, still surfacing failures that reveal new layers, still being shaped by the work it's doing.

---

## The forward experiment

The Knowledge Compounding Protocol currently runs on one agent. Extending it to the Outreach Drafter — which produces a measurable outcome every time it drafts a message — is the near-term test of whether the protocol generalizes beyond Claude Code. If it does, Layer 4 is mechanism, not Claude-Code-specific accident. If it doesn't, the protocol's scope narrows and the case study revises in that direction.

That result is due within six weeks of publication. This README will be updated with the outcome.

---

*Published April 2026. Case study authored by Jeff Millett. Repo maintained by the same operator. The architecture diagram is finished; the architecture is not.*
