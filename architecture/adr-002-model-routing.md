# ADR-002: Multi-Provider Maker-Checker Routing

**Status:** Accepted (April 22, 2026)
**Author:** Jeff Millett
**Category:** Architecture / Agent Fleet
**Related:** ADR-001 (Knowledge Compounding Protocol), Operator Profile

---

## Context

By mid-April 2026 the agent fleet was nine agents deep. Strategy, Product, CC, and PRR Tracker were running active operational work. Outreach Drafter, QA Triage, Data Discovery, and Deliverable QA were skill-defined and queued for activation. All of them ran on Opus 4.6 by default — that was the model the chats were opened with, and nobody had asked which agents needed the frontier model and which didn't.

Three pressures showed up in the same week.

**One: cost.** Running every agent on the most expensive Claude tier was wasteful for the work most of them did. PRR follow-up drafting, Intel Queue extraction, QA Triage classification — these are bounded structured tasks with well-defined input/output schemas. The frontier model wasn't earning its premium on them.

**Two: a real failure.** On April 2, Codex (OpenAI's repo-aware review agent, run from chatgpt.com/codex against the deltascanner GitHub repo) audited a set of standing rules that Strategy chat had drafted the day before. The rules referenced four file paths that did not exist in the repository. Strategy chat had written the rules from the spec, not from the deployed codebase, and the reviewing pass had not caught the gap because Strategy chat WAS the reviewing pass. Codex caught it in five minutes. That was a same-family review failure: builder-evaluating-own-work, exactly the failure mode the Anthropic Nov 2025 harness post warned about — builders who evaluate their own work are systematically overoptimistic.

**Three: speed.** Some agents needed to run cheap and fast, not careful and deep. Data Discovery scanning municipal open-data portals for code-enforcement schemas is commodity reconnaissance — listing what exists, naming what fields it has. The work doesn't need deep reasoning. It needs throughput.

The three pressures pointed in the same direction: stop running everything at the same tier, and stop using the same family for both build and check.

---

## Decision

I split the agent fleet into three tiers and added an external checker on a different provider.

**Tier 1 — Frontier maker (Claude Opus 4.6).** Strategy chat, Product chat, Claude Code (CC). These produce specs, architectural decisions, code, and standing rules. Their outputs feed every other agent and every shipped build. Errors here cascade downstream. They run on the best model I have access to.

**Tier 2 — Structured execution (Claude Sonnet 4.6).** PRR Tracker, CC Intel Queue, QA Triage, Outreach Drafter, Deliverable QA. These do bounded work with defined schemas — sweeping the inbox for record-request responses, classifying intel insights into the YAML scorer, drafting outreach DMs against templates, running mechanical QA checks. They benefit from the maker-family training on instruction-following and structured output but don't need frontier reasoning.

**Tier 3 — Commodity research (GPT-4o-mini).** Data Discovery. Scanning open-data portals, listing datasets, comparing field schemas across municipalities. The work is high-volume, low-judgment, well-structured. GPT-4o-mini is the first non-Anthropic agent in the fleet and is roughly 20-40x cheaper than Opus on public token pricing.

**External checker — Codex (OpenAI, GPT-5 family).** Triggered automatically after any build that modifies scoring architecture, shared modules, config schemas, or standing rules. Codex reads the actual repo state, not the spec, and reports back whether the build matches what was documented. It runs on a different family than every other Tier-1 maker, so its review is structurally independent.

The fleet also has an on-demand model-diversity escape hatch: ChatGPT can be invoked for "second opinion" passes when I want a non-Anthropic read on a strategic call, or for structured-schema validation against live municipal endpoints (where it has tested better than Claude in my workflow). This is not a gated tier — it's a tool I reach for when the question warrants it.

Every agent declares its model + provider in `wikis/agent-os/AGENTS.md`. The declaration is the contract. Re-pointing an agent to a different model is a one-line edit in the registry plus a corresponding change in the agent's chat or skill.

---

## Alternatives considered

**1. Single-model fleet (all Opus, no separation).** Status quo before April 22. Rejected because the cost-quality tradeoff was bad — the fleet was paying frontier prices for commodity work — and because keeping everything on one family meant the structural condition for catching same-family blind spots was missing. The April 2 Codex incident wasn't a one-time accident waiting to be filed away; it was the visible surface of the deeper problem. Same-family review systematically misses errors that the same family also produced. Doing more careful Anthropic-only review would not have caught the four nonexistent file paths, because the source of the error was Anthropic-only review.

**2. Persona separation within one model.** One chat, different role-plays. "First reason as the architect, then reason as the skeptic." Rejected because persona separation inside a single context window doesn't break the prior — the persona reads the same context, draws on the same training, lands on the same conclusions. I had been doing this implicitly for months (Strategy chat self-critiquing its own deliverables before shipping). It was not catching the things Codex caught.

**3. Same-family review with different prompts.** Multiple Anthropic chats with different system prompts — one chat builds, another chat reviews against an adversarial prompt. Rejected for the same root reason as alternative 2 plus an additional structural concern: Anthropic models share training data and architecture. Even with adversarial prompting, the failure modes correlate. The April 2 catch was not "Codex was prompted more carefully" — it was "Codex is built on a different family." Same-family review with better prompts is a more expensive version of self-review.

**4. Fully manual external review (I am the checker).** Skip automated/external review entirely; I review everything myself before anything ships. Rejected on throughput grounds. The fleet's purpose is to remove me from operational loops. Making me the gating reviewer for every spec, every build, every standing rule reverses the direction. It also doesn't solve the maker-checker problem — it just substitutes one same-family reviewer (Anthropic chat) for another (me). I'm not less correlated with Anthropic-built outputs than another Anthropic chat is. I read the same docs. I anchor on the same examples. I miss the same things.

**5. Drop the skeptic review entirely.** No external audit layer. Trust the maker. Rejected because the April 2 incident measured the cost of doing this. Four nonexistent file paths in a published standing-rules document is a credibility problem if a buyer ever sees it, and a discipline problem regardless. The skeptic review is cheap (Codex runs in five minutes) and the failure cost compounds. Removing the only layer that catches a known failure mode to save five minutes is not a tradeoff I'd take.

---

## Consequences

**Positive.**
- Cost ceiling controlled per agent. The fleet's monthly model spend is bounded by routing, not by chat volume.
- Errors that same-family review misses are caught by a different family before they ship. The April 2 catch is the signal; Codex now runs as an automatic gate on the build types most likely to generate cascading errors.
- The model declaration in AGENTS.md is a discipline artifact. When I re-point an agent, I have to write down why. The registry forces the routing decision to be explicit.
- The fleet can absorb new agents without re-litigating the routing. New skill → declare its tier → done.

**Negative.**
- More vendors, more credentials, more failure modes. OpenAI billing is a separate account. Codex is a separate workflow. ChatGPT is a separate paste relay for ad-hoc work. The provider sprawl is real overhead.
- The provider-abstraction layer is partially shipped. AGENTS.md declares the model; the resolver is manual v1 (I open the correct chat). There's no config-driven dispatcher routing prompts to the right provider — that work is gated behind needing it badly enough to build it.
- The Sonnet/Opus split is a structural prior, not a measured calibration. I haven't run the controlled experiment "Sonnet on identical structured tasks vs Opus, scored on a binary quality check." The split is grounded in published model-card claims and intuition about task complexity. It is not validated against my own data yet.
- Codex's failure-detection rate is observed at n=1. The April 2 catch is the data point I'm extrapolating from. I don't yet know what Codex's false-positive rate is on routine builds, or whether the catch-rate sustains across repo changes Codex has been trained less directly on.

---

## Honest limits

This decision rests on four claims I cannot yet fully verify.

**The maker-checker principle is borrowed.** The source is Anthropic's Nov 2025 harness post: builders who evaluate their own work are systematically overoptimistic. Externally well-grounded — Anthropic was reporting on their own internal eval data. Internally, I have one catch (April 2) and one structural argument. I do not have a controlled experiment running same-family review against different-family review on identical specs. The claim might be load-bearing on a single anecdote.

**Cross-family error correlation is unmeasured.** GPT and Claude are trained on overlapping data. The structural assumption behind different-family review is that they fail differently. Maybe they do; maybe they fail in correlated ways on the failure modes I most care about (file-path verification, schema accuracy, spec-vs-deploy drift). I don't have the data. The April 2 catch is consistent with the structural hypothesis but doesn't prove it.

**The cost claim comes from public pricing, not measured fleet spend.** "20-40x cheaper" for GPT-4o-mini comes from token-pricing math, not from monthly bills observed across the fleet. The actual cost ratio depends on prompt size, output size, retry rates, and whether the cheap model needs more turns to complete tasks the expensive model would solve in one. I'll know in 30-60 days of operation whether the ratio holds at the workflow level.

**The Sonnet tier is a structural prior, not a calibrated decision.** I have not measured Sonnet's reliability on the specific structured-task workloads I assigned it (PRR Tracker, Intel Queue, QA Triage, Outreach Drafter, Deliverable QA) against Opus on the same workloads. The calibration test would be: run identical inputs through both tiers for two weeks, score outputs against the same binary quality checks, compare reject rates. I haven't built that harness. Until I do, the tier split is an informed guess.

---

## Revisit triggers

I revisit this decision if any of the following hold:

1. **External-audit signal degrades.** Codex's false-positive rate exceeds 30% over its next 10 audits, or it misses a structural error that a same-family review caught afterward. Either invalidates the cross-family premise.

2. **Cost claim doesn't hold at workflow level.** After 30 days of Tier-3 operation, measured monthly cost is not materially below what equivalent Opus usage would have been. The cost premise was the partial reason for the tier split; if the premise fails, the tier collapses back into Anthropic-only.

3. **Sonnet-tier quality gap surfaces.** A Sonnet-tier agent produces a measurable quality drop versus Opus on identical work — caught either by Jeff's review of agent outputs or by a downstream error trace. That agent gets upgraded; if the gap is fleet-wide, the tier mapping collapses.

4. **The model surface changes underneath.** Anthropic ships a future Sonnet that matches Opus on high-stakes work, or a future Opus that's cheap enough that the cost-driven tier split stops mattering. The tier mapping gets re-derived from the new pricing-and-capability surface, not preserved out of inertia.

---

## Notes

This ADR documents a decision shipped on April 22, 2026 as part of S15 (Multi-Provider Architecture) and S14 (Agent OS scaffolding). Both decisions resolved in the same session because they were structurally linked: the Agent OS registry (`AGENTS.md`) is the surface that makes multi-provider routing tractable. Routing without a registry is invisible. The registry without routing is a list.

The decision interacts with ADR-001 (Knowledge Compounding Protocol). ADR-001 makes solution files the cross-session memory layer for builds; ADR-002 makes the model that writes those solution files different from the model that audits them. Both decisions trace to the same root pattern: structural separation between phases of work catches errors that better effort within one phase does not.

---

*ADR-002 | Decided April 22, 2026 | Documented April 26, 2026*
