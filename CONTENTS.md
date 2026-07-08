# Repository contents

Everything below is the verification layer for the [README](./README.md) pitch. Every mechanism named there has a corresponding file here.

## Case studies

- [`case-studies/agent-operator.md`](./case-studies/agent-operator.md) — *How I operate the agents that build.* Captured 2026-05-08. A four-hour session retrofitting three private repos onto a unified architecture; the moment a 1,205-line commit was almost discarded as drift before an operator-side gate reversed the call.
- Architecture case study (long-form, ~3,500 words) — *What was built and the failures that produced each layer.* Available on request; compressed version at [`one-pagers/case-study-onepager.md`](./one-pagers/case-study-onepager.md).

## Architecture diagram

![Deltascanner agent fleet — five-layer architecture](./architecture/fleet-diagram.svg)

## One-pagers (60-second reads)

- [`case-study-onepager.md`](./one-pagers/case-study-onepager.md) — The five-layer architecture, top-down.
- [`model-comparison-onepager.md`](./one-pagers/model-comparison-onepager.md) — *"The Layer Nobody Designs For."* The 36-hour pattern that surfaced the Operator Profile.
- [`adr-001-onepager.md`](./one-pagers/adr-001-onepager.md) — Knowledge Compounding Protocol.
- [`adr-002-onepager.md`](./one-pagers/adr-002-onepager.md) — Multi-Provider Maker-Checker Routing.
- [`adr-003-onepager.md`](./one-pagers/adr-003-onepager.md) — Scoring architecture: centralized config in production; universal-scorer consolidation built but not shipped.
- [`adr-004-onepager.md`](./one-pagers/adr-004-onepager.md) — Per-Agent Governance Tiers with Batched Review.
- [`adr-005-onepager.md`](./one-pagers/adr-005-onepager.md) — Ban Recency in Distress Scoring.
- [`operator-profile-onepager.md`](./one-pagers/operator-profile-onepager.md) — The Layer 1 calibration file.

## Compounded intelligence

- [`briefs/ai-pm-brief.md`](./briefs/ai-pm-brief.md) — Six principles compounded across thirteen batches of intelligence extraction; cross-references to public sources (Cat Wu, Mahesh Yadav, others) that converged independently on the same operating model.

## Supporting artifacts

- [`handoff-template.md`](./handoff-template.md) — The structured format every agent uses to close a session.
- [`examples/cc-spec-standards.md`](./examples/cc-spec-standards.md) — The standard for every build spec destined for Claude Code. One of the four changes that moved first-pass build acceptance from 31% to 86%.
- [`examples/spec-prod30-atomic-pipeline.md`](./examples/spec-prod30-atomic-pipeline.md) — Representative build spec; the PROD30 build is the site of a pipeline-gate false positive documented in the solution files.
- [`examples/solution-files/`](./examples/solution-files/) — Three structured solution files from the Knowledge Compounding Protocol, each encoding a failure class caught in April 2026.
- [`examples/operator-profile.md`](./examples/operator-profile.md) — Sanitized v5 of the Layer 1 calibration file. v1→v5 changelog, April 25–27, 2026.
- [`agent-fleet.md`](./agent-fleet.md) — Per-agent breakdown of the 8-agent fleet: scope, boundaries, handoff format, escalation paths, version history.
- [`metrics/gate-metrics.md`](./metrics/gate-metrics.md) — Forward-logging template for first-pass acceptance rate and 30-day defect traceback. Keeps the gate-effectiveness claim falsifiable.

## Architecture Decision Records

- [`architecture/adr-001-knowledge-compounding-protocol.md`](./architecture/adr-001-knowledge-compounding-protocol.md)
- [`architecture/adr-002-model-routing.md`](./architecture/adr-002-model-routing.md)
- [`architecture/adr-003-scoring-architecture.md`](./architecture/adr-003-scoring-architecture.md)
- [`architecture/adr-004-curator-mode-batched-review.md`](./architecture/adr-004-curator-mode-batched-review.md)
- [`architecture/adr-005-recency-ban.md`](./architecture/adr-005-recency-ban.md)

## The forward experiment

The Knowledge Compounding Protocol currently runs on one agent. Extending it to the Outreach Drafter — which produces a measurable outcome every time it drafts a message — is the near-term test of whether the protocol generalizes beyond Claude Code. Due mid-June 2026; this file will be updated with the outcome.
