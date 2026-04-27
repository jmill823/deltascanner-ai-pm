# Agent Fleet

The platform is operated by a fleet of nine bounded agents, each governed by an explicit skill file. This page is the layer-3 expansion of the operator profile and ADR-004: it covers what each agent does, what it explicitly does *not* do, how its output flows downstream, when it escalates, and how it has changed over time.

The fleet was designed under one constraint: **the founder is the reviewer, not the router.** Each agent's scope is defined to make that constraint hold — agents that touch customer-facing surfaces require approval before action; agents that operate inside controlled bounds run autonomously and surface their work for batched review. ADR-004 covers the tier architecture; this page covers the agents.

## At a glance

| Agent | Tier | Version | Primary role |
|---|---|---|---|
| Strategy | Gate | v1 | Reconciliation hub, doctrine-bounded synthesis, spec authoring |
| Product | Gate | v4 | CC build spec authoring with acceptance tests |
| Outreach Drafter | Gate | v5 | Outreach DM/email drafting; gift-first second-touch framework |
| PRR Tracker | Reviewer | v5 | Public-records-request operations; weekly digest |
| CC Intel Queue | Reviewer | v3 | Per-email insight extraction; compile instructions for repo writes |
| QA Triage | Reviewer | v4 | Three-lane QA categorization |
| Data Discovery | Reviewer | v2 | City research for tax delinquency / code enforcement sources |
| Deliverable QA | Reviewer | v1 | Buyer-facing PDF/document review |
| CC Spec Standards | (standards doc) | v1 | Spec template that governs every Claude Code build |

Tier definitions are in ADR-004. Below, each agent is expanded.

---

## Strategy Agent (v1)

**Scope.** Reconciliation hub for all other agents. Reads handoffs from agent sessions and merges them into project knowledge. Authors specs that go to Claude Code. Synthesizes across silos when an architectural question crosses agent boundaries. Runs the doctrine-check + contradiction-gate protocols before shipping any architectural proposal (per the four standing rules added April 19, 2026).

**What it does NOT do.** It doesn't decide whether to build something — that's a founder call after reviewing an intake card. It doesn't choose pricing, prospect targeting, or visual design choices. It doesn't write Claude Code implementation details — CC decides those within spec boundaries. It doesn't update Notion, send DMs, or process PRR inboxes.

**Handoff format.** Closes every session that produces changes with a handoff document mapping each change to a specific project knowledge file and naming the specific edit. "Update PIPELINE" is rejected; "DS-CITY-PIPELINE.md — move Tampa from 'In Progress' to 'Pipeline-Ready'" is the standard. Strategy learnings appended to `ds-strategy-learnings.md` at session close.

**Escalation paths.** Architectural proposals → contradiction gate (does this contradict standing doctrine? if yes, resolve before shipping). Items that cross agent ownership boundaries → name the operator/orchestrator/reviewer explicitly before designing. Any decision the founder hasn't approved → surface to the founder rather than ship.

**Version history.** v1 (April 3, 2026): initial skill file with reconciliation protocol, multi-file production rules, integration-with-other-agents matrix. Four behavior rules for architectural proposals added April 19 after a substrate-miscalibration incident exposed that fresh-context synthesis was bypassing standing doctrine.

---

## Product Agent (v4)

**Scope.** Authors Claude Code build specs from intake cards. Maintains the intake card → feature spec workflow. Runs the workflow regression check (six core user flows traced for any change). Detects when a build needs Codex external review and prompts the founder before deploy. Maintains the scope creep guard — every spec includes a "What NOT To Do" section populated from the intake card's scope boundary.

**What it does NOT do.** It doesn't execute builds — Claude Code does. It doesn't make scoring weight decisions — the Strategy chat does, drawing from the scoring registry. It doesn't write customer-facing copy. It doesn't decide priority — the founder does in the TODO pass.

**Handoff format.** Feature specs follow `cc-spec-standards.md` (Block 7 acceptance tests required; specs without acceptance tests are incomplete and don't ship). Closes builds with a deploy report referencing actual repo paths. Scope creep signals (e.g., a UI change that requires a pipeline change to support it) get flagged, not silently expanded.

**Escalation paths.** Effort estimate jumps from S to L during spec writing → resurface to founder. Scoring/architecture/config build → trigger Codex review prompt before deploy. Standing rule modifications → Strategy chat reconciles.

**Version history.** v3 (April 2, 2026): Codex review prompt-me trigger added after the L2 refactor exposed documentation-vs-code drift. v4 (April 3, 2026): two-tier validation (Block 6 mechanical + Block 7 acceptance tests) and scope creep guard added.

---

## Outreach Drafter Agent (v5)

**Scope.** Drafts outreach DMs and emails. Touch 1, Touch 2 (gift-first framework — opens with a specific concrete item that demonstrates the product's output rather than describing the product), and follow-ups. Sales-kit-aware: pulls language from the sales kit's objection handling and buyer-persona sections rather than improvising. Enforces join-language protection — describes outputs and the gap they fill, never describes inputs.

**What it does NOT do.** It doesn't send — every draft awaits founder approval. It doesn't choose prospects — the founder does, with the warm-leads tracker as input. It doesn't quote pricing before the prospect has experienced the product (sales motion rule). It doesn't author cold subject lines or sequencing logic — that's a separate decision layer.

**Handoff format.** Drafts surface in the chat ready to copy. Batched delivery (multiple drafts at once) for review during Tuesday/Thursday check-ins. Each draft tagged with prospect name, touch number, and intended channel.

**Escalation paths.** Standing-rule violation in a draft (join-language slip, pricing quote without product experience) → halt and flag rather than ship. Prospect responds with a buy-box request → kick to Strategy chat for buy-box curation, not auto-handle.

**Version history.** v3 (March 2026): join-language protection added after a draft surfaced product-input language that would have leaked the data architecture. v4 (April 2026): gift-first second-touch framework replaced generic follow-up language. v5 (April 24, 2026): wiki-native, integrated with the agent-fleet wiki layer.

---

## PRR Tracker Agent (v5)

**Scope.** Operates on the PRR backlog living at `wikis/prr/` in the cross-project agent OS repo. Inbox sweeps to triage incoming records-office responses. Drafts new requests and follow-ups. Auto-sends routine follow-ups to known coordinators. Produces a weekly digest as the heartbeat — silence in the digest is the system-broken signal.

**What it does NOT do.** It doesn't auto-send first-contact requests to new agencies (founder approves). It doesn't auto-send substantive replies, appeals, or untested-channel attempts (founder approves). It doesn't decide whether a denial is worth appealing (founder + cost-benefit). It doesn't choose which cities to file PRRs for — that's a Strategy chat decision based on pipeline priorities.

**Handoff format.** Weekly digest in `wikis/prr/_digest.md`. Same-day chat notifications for high-signal events (denials, cost estimates, data-received emails, kill-signal candidates). Routine auto-acks suppressed from notifications to keep the surface clean.

**Escalation paths.** Cost estimate received → founder decides on payment dispatch. Denial received → founder decides on appeal. Three+ auto-acks without substantive response from a single agency → flag as "stuck" signal.

**Version history.** v1–v4 covered iterative refinement of the digest format and notification thresholds. v5 (April 24, 2026): wiki-native, git-versioned, became the prototype for the broader hybrid wiki architecture used by all reviewer-tier agents.

---

## CC Intel Queue (v3)

**Scope.** Processes self-sent emails containing build lessons, market signals, competitor intel, and tool discoveries. Extracts each email into a structured raw record (`raw/[id].md` with YAML frontmatter for project tags, ROI, effort, status, one-line summary, body). Generates compile instructions for Claude Code to write to the intel repo — never writes directly. Identifies cross-project connections that the email's project prefix wouldn't surface.

**What it does NOT do.** It doesn't make strategic decisions — those happen in Strategy chat. It doesn't update DS-TODO, DS-PLAYBOOK, or any project file directly. It doesn't write specs. It doesn't draft outreach. It doesn't process PRR emails. It doesn't commit to the intel repo (Claude Code does that).

**Handoff format.** Compile instructions packaged as a single CC-ready prompt: raw file content, wiki page updates, INTEL-INDEX.md row, OVERVIEW.md update. Founder pastes to Claude Code in the intel folder; CC creates files, updates wikis, runs acceptance tests, commits, pushes.

**Escalation paths.** Insight contradicts an existing wiki principle → flag as a contradiction for Strategy chat to reconcile, don't silently overwrite. Cross-project connection that the email's prefix wouldn't surface → produce in the "Not On Your Radar" section of the batch.

**Version history.** v1 (March 2026): initial extraction protocol. v2 (April 2026): cross-project tagging added after evidence that single-project tagging was missing patterns visible across silos. v3 (April 22, 2026): three-layer knowledge doctrine codified — `raw/` (firehose), `briefs/` (project-aggregated), `dashboard/data/` (curated/scored). Intel Queue writes to `raw/` only.

---

## QA Triage Agent (v4)

**Scope.** Categorizes site QA tickets into three lanes. Lane 1 = mechanical fixes batched for the next Claude Code session. Lane 2 = needs a spec, queued for the next intake card. Lane 3 = parked or blocked, logged with the reason.

**What it does NOT do.** It doesn't fix issues — Claude Code does. It doesn't run regression testing — the founder verifies on a physical device after the fix. It doesn't make priority calls between lanes — the lane assignment is the priority.

**Handoff format.** Lane 1 → Claude Code as a batched ticket. Lane 2 → Strategy chat spec queue. Lane 3 → log with reason (e.g., "blocked on PRR data," "depends on city refresh").

**Escalation paths.** Cross-cutting issue affecting multiple cities → kicks to Strategy chat. Issue that suggests a regression pattern not yet documented → flag for inclusion in `cc-spec-standards.md` known regressions.

**Version history.** v1–v2 (March 2026): initial three-lane scheme. v3 (April 2026): division of labor between mechanical CC fixes and ad-hoc founder verification. v4 (April 24, 2026): batch delivery pattern formalized for Tuesday/Thursday check-in cadence.

---

## Data Discovery (v2)

**Scope.** City-by-city research on tax delinquency and code enforcement data sources — portal availability, Socrata datasets, PRR strategy, residency restrictions, file format expectations. Produces discovery reports that feed into intake cards. Currently the only non-Anthropic agent in the fleet, running on `gpt-4o-mini` for cost optimization on bulk research tasks.

**What it does NOT do.** It doesn't execute PRR filings — PRR Tracker does. It doesn't make build decisions — Strategy/Product chat does. It doesn't validate data quality on received files — Stage 2 validation handles that.

**Handoff format.** Discovery report per target city, structured as: data sources identified, retrieval strategy, known restrictions, effort estimate, open questions.

**Escalation paths.** Legal feasibility issue surfaces (residency restriction, statutory limit) → kicks to Strategy chat. Data source not previously characterized in any city → flag for canonicalization in the city-research playbook.

**Version history.** v1 (March 2026): initial research prompts. v2 (April 2026): bulk research workflows added; Socrata Discovery API integration tested.

---

## Deliverable QA (v1)

**Scope.** Reviews buyer-facing PDFs and documents before they ship. Brand consistency check (colors, typography, logo placement). Sales-kit alignment (language matches the buyer persona's vocabulary, objection handling matches the kit). Tone match against the operator profile.

**What it does NOT do.** It doesn't author content — that's Outreach Drafter or the founder. It doesn't choose recipients. It doesn't override the founder's judgment on a deliverable that passed sales-kit review.

**Handoff format.** Approved deliverable with QA notes attached → Outreach Drafter for delivery, or directly to the founder for hand-off.

**Escalation paths.** Sales-kit drift detected (a deliverable repeatedly contradicts current sales-kit framing) → kicks to Strategy chat for sales-kit refresh.

**Version history.** v1 (April 24, 2026): initial skill file. Activates per-deliverable; no dedicated chat yet.

---

## CC Spec Standards (v1)

**Note.** Not an agent — a standards document that governs every Claude Code build spec. Included here because it sits at the same boundary as the agents and shapes the spec → execute → review handoff that defines the build flow.

**Scope.** Defines the structure every CC build spec must follow: scope statement, files affected, implementation notes, what-NOT-to-do section, verification checklist, deviation rules, Block 6 post-deploy QA, Block 7 acceptance tests. Standardizes the contract between spec authors (Product/Strategy chats) and the build executor (Claude Code).

**What it does NOT do.** It doesn't write specs — Product Agent and Strategy chat do. It doesn't execute builds. It doesn't decide whether a spec is ready — the acceptance-test-required rule (April 3, 2026) does that automatically: a spec without acceptance tests is incomplete and doesn't ship.

**Handoff format.** Specs that conform to the standard go to Claude Code. Specs that don't conform are returned to the spec author for completion before send.

**Escalation paths.** Spec author needs to deviate from a standard pattern → document in the spec, justify, proceed. Repeated deviations of the same type → revise the standard rather than accumulate exceptions.

**Version history.** v1 (April 3, 2026): consolidated from prior CC-SPEC-TEMPLATE-v1 and v2, with Block 7 acceptance tests added as a hard requirement.

---

## How the fleet integrates

The fleet operates as a graph, not a hierarchy. Strategy is the reconciliation hub but not a manager — it reads handoffs from peer agents and merges them into project knowledge. Each agent owns its own wiki section in `/wikis/[agent]/`; cross-cutting work happens through `/wikis/shared/` files where multiple agents contribute to a single entity (a city, a prospect, a pattern) under contract-governed section ownership.

The integration matrix is documented in the Strategy Agent skill — what Strategy does with each peer agent's output, and where each output lands in project knowledge. The general principle: every agent writes to its own surface, reads from shared surfaces, and surfaces work to the founder at the cadence its tier dictates. Cross-pollination happens through the shared surfaces, not through agent-to-agent direct calls.
