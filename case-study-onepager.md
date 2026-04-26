# DeltaScanner as an AI Product Management Case Study

A model upgrade broke my agent system last week. The fix was a file I'd never thought to write — an [Operator Profile](./examples/operator-profile.md) encoding how I work, model-agnostic, ported across the fleet. The layer didn't appear in any architecture diagram I'd seen. That's the case study. **The architecture wasn't designed; it was discovered through operating.**

I'm a non-technical solo founder. I do not write build code. The agents write, build, deploy, and surface work for review. I approve, reject, or defer at scheduled gates. Between gates, my inbox is empty by design. That's not a workflow choice — it's a structural property the system is engineered to maintain. Every layer in the stack exists because something specific broke and I encoded the fix.

## The architecture, top-down

*Layer 1 — Interaction.* Operator Profile (calibration contract, model-agnostic) plus the product surface (`deltascanner.com`). Origin: a model upgrade exposed that the calibration layer was implicit, not explicit, and shifted with the model's defaults.

*Layer 2 — Orchestration.* Strategy chat as the routing layer. Curator-mode review, every handoff a [contract](./handoff-template.md), every spec required to include reproduction steps, acceptance tests, post-deploy QA, scope boundary. Origin: three substrate miscalibrations in one session and a queue-backup failure mode that degrades the operator into a rubber stamp.

*Layer 3 — Specialized Agents.* Eight agents with single responsibilities and versioned skill files: PRR Tracker, Outreach Drafter, QA Triage, Product, Strategy, Claude Code (build), Data Discovery, Deliverable QA. Origin: each agent accreted because a class of work needed its own lane and quality standard.

*Layer 4 — Data.* The [Knowledge Compounding Protocol](./examples/solution-files/) — Claude Code writes a structured solution file after every build, reads all prior files before every new build, surfaces contradictions as BLOCKING flags. 36 files seeded. Origin: cross-session knowledge was 100% dark; every build paid the cost of every prior build's discoveries.

*Layer 5 — Model API.* Maker-checker separation across four models: Claude (strategy), Claude Code (build), Perplexity (discovery), ChatGPT/Codex (adversarial review). Origin: Codex caught a Claude-Code-authored handoff that documented a non-existent architecture. The same-model-as-its-own-auditor failure isn't hypothetical.

## Evidence

A March 30 architecture audit revealed 8 defects across 13 live market models — scoring components shouldn't have been active, weight drift, architecture labels that didn't match actual behavior. The structural response was four-part: a YAML-configured universal scorer, schema validation, [acceptance test standards](./examples/cc-spec-standards.md) required in every spec before build code, and a Codex external review gate.

Before: 4 of 13 builds passed first-pass acceptance (31%).
After: 6 of 7 builds passed first-pass acceptance (~86%).

Hialeah, Tampa, West Palm Beach, Los Angeles, and additional cities close in the next six weeks. The methodology is tested on a growing build set, not a frozen retrospective.

## What this evidence does not prove

Four structural changes co-occurred. I can't causally isolate which produced most of the rate gain. The Goodhart risk on gate metrics is mitigated — not eliminated — by complexity-weighted build counts, retroactive defect traceback, and reject-reason categorization. My attention is the enforcement layer for the safeguards. Scaling this past me requires moving that enforcement into the protocol itself, which I haven't specified yet.

## What I'm measuring next

The Knowledge Compounding Protocol currently runs on one agent. Extending it to the Outreach Drafter — which produces a measurable outcome every time it drafts a message — is the near-term test of whether structured cross-session memory generalizes beyond Claude Code. If it does, Layer 4 is mechanism, not Claude-Code-specific accident. If it doesn't, the protocol's scope narrows and the case study revises in that direction.

---

**Full case study available** in [`case-study-full.md`](./case-study-full.md). It walks each layer in depth, names the specific failures that produced each one, presents the metrics the system is measured against, and surfaces the experiments running now that would change my mind about the thesis.
