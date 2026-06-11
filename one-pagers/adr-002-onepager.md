# ADR-002: Multi-Provider Maker-Checker Routing (one-pager)

**Accepted April 22, 2026** | Architecture / Agent Fleet

## Context

Nine agents, all on Opus 4.6 by default — because that's what the chats opened with. Three pressures arrived the same week: cost (frontier prices for commodity work), a real failure (Codex audited a Strategy-chat-authored standing rules document and caught four file paths that didn't exist; same-family review had missed it because Strategy chat *was* the reviewer), and speed (some agents needed throughput, not depth).

## Decision

Three tiers plus an external checker on a different provider.

- **Tier 1 — Frontier maker (Claude Opus 4.6):** Strategy, Product, Claude Code. Errors here cascade.
- **Tier 2 — Structured execution (Claude Sonnet 4.6):** PRR Tracker, Intel Queue, QA Triage, Outreach Drafter, Deliverable QA. Bounded schemas, well-defined I/O.
- **Tier 3 — Commodity research (GPT-4o-mini):** Data Discovery. High-volume, low-judgment, structured.
- **External checker — Codex (OpenAI, GPT-5 family):** Auto-triggered on builds modifying scoring, shared modules, schemas, or standing rules. Different family. Reads repo state, not spec.

Each agent declares its model + provider in `wikis/agent-os/AGENTS.md`. The declaration is the contract.

## Alternatives (one line each)

Single-model fleet — bad cost-quality tradeoff plus same-family blind spot. Persona separation in one model — same context, same training, same conclusions. Same-family review with adversarial prompts — failure modes correlate at the family level. Manual external review (operator as checker) — reverses the system's purpose. Drop skeptic review entirely — five minutes of Codex caught what would have shipped as a credibility problem.

## Honest limit

The maker-checker principle is borrowed from Anthropic's Nov 2025 harness post; I have one catch (April 2) and one structural argument — not a controlled experiment. Cross-family error correlation is unmeasured. The Sonnet/Opus split is a structural prior grounded in published model-card claims, not calibrated against my own data. The cost claim comes from public token pricing, not measured fleet spend.

## Revisit triggers

Codex's false-positive rate exceeds 30% over its next 10 audits. Measured monthly cost is not materially below equivalent Opus usage after 30 days of Tier-3 operation. A Sonnet-tier agent produces a measurable quality drop versus Opus on identical work.

---

**Full ADR** at [`architecture/adr-002-model-routing.md`](../architecture/adr-002-model-routing.md). Details five rejected alternatives, four claims this decision rests on but cannot yet verify, and the registry-as-contract pattern that interacts with ADR-001.
