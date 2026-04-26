# Agent Handoff Template

A structured format for closing any agent session that produced changes. The template exists so a Strategy agent (or a human operator acting as one) can reconcile work from multiple parallel agents without rebuilding context or interpreting prose.

## Where this fits in the case study

This template is the concrete form of two claims the case study makes in Layer 2 (Orchestration) and Layer 4 (Data):

- *Every handoff between agents follows a contract.* The handoff is the contract. A handoff that's missing a section is rejected at the gate before downstream work happens. This is what makes batched review fast — the operator scans a known structure for known fields in 30-60 seconds, not prose for meaning.
- *Knowledge compounds across sessions.* The Session Learnings section is the write-step of the Knowledge Compounding Protocol at the agent level. Each agent captures what it learned during a session. The Strategy agent appends those learnings to per-agent learnings files. The next session's agent reads its own learnings file before starting.

Without the first property, the curator degrades into a rubber stamp when the queue backs up. Without the second, every session starts from zero and prior lessons are lost.

## Adaptation notes

This is the DeltaScanner version. The agent names (Outreach Drafter, PRR Tracker, QA Triage, Product, Strategy, CC Intel Queue, Data Discovery, Deliverable QA) and the project file names (DS-TODO, DS-PLAYBOOK, DS-CITY-PIPELINE, DS-SCORING-REGISTRY, DS-SALES-KIT, DS-OPS-PLAYBOOK) are specific to the fleet documented in the case study. If you're adapting this template:

- Replace DS-specific file names with your own project's canonical files
- Adjust the agent-specific learnings tables to match your fleet's agents
- Keep the structural sections (What Happened / Cascade Updates / Open Decisions / Files Produced / Session Learnings / Specs Ready) — these are the contract

---

## Why This Exists (original rationale)

The Strategy Agent reconciles handoffs from other agents into project files. Reconciliation quality depends on handoff quality. When a handoff explicitly maps changes to project files, reconciliation is mechanical. When it doesn't, the Strategy Agent interprets — and interpretation is where shortcuts and missed items happen.

This template ensures every handoff arrives in a format the Strategy Agent can process without guessing.

---

## Template

Every agent session that produces changes should close with a handoff using this structure. Copy this template, fill it in, and save as `[agent]-handoff-[YYYY-MM-DD].md`.

```
# [AGENT NAME] SESSION HANDOFF — [Date]
### For Strategy Chat Reconciliation

---

## Session Type: [Long / Short / Sporadic]

---

## What Happened This Session

### 1. [First major item — descriptive title]
[2-5 sentences: what was done, what the outcome was, what decisions were made]

### 2. [Second major item]
[Same format]

### 3. [Continue as needed]

---

## Cascade Updates Needed (Strategy Chat)

Map every change to its destination file. If a file isn't affected, don't include it.

### DS-TODO.md
- Add: [specific task with enough detail to write the TODO entry]
- Update: [specific item and what changed]
- Close: [specific item — mark done with result]
- Remove: [specific item — why]

### DS-PLAYBOOK.md
- [Specific change — new standing rule, status update, decision resolved]

### DS-CITY-PIPELINE.md
- [Specific change — city status, data received, build progress]

### DS-SCORING-REGISTRY.md
- [Specific change — or "No change until [condition]"]

### DS-SALES-KIT.md
- [Specific change — prospect update, objection, positioning]

### CC-LESSONS-LEARNED.md
- [ARCHIVED — active lessons now live in `docs/solutions/` in the repo. Only include here if a lesson needs to be seeded before CC runs the Knowledge Compounding Protocol build.]

### DS-OPS-PLAYBOOK.md
- [Specific change — or omit if none]

### Agent Skill Files
- [Version bump, new rule, template change — or omit if none]

### Notion Trackers
- PRR Tracker: [Status changes with row identifiers]
- Outreach Tracker: [New rows, status changes, touch dates]

---

## Open Decisions for Jeff

| # | Decision | Options | Urgency |
|---|---|---|---|
| 1 | [What needs deciding] | [The choices] | [High / Medium / Low] |

---

## Files Produced This Session

| File | Purpose | Destination |
|---|---|---|
| [filename] | [what it is] | [project knowledge / CC / outputs] |

---

## Session Learnings

What did this session teach that would make the next session better?
Fill in the fields relevant to your agent type. If nothing was learned, write "Clean session — no new learnings."

### Outreach Drafter Learnings
| Field | Value |
|---|---|
| Prospect profile(s) touched | [role, market, certification — what type of person] |
| Hook/approach used | [Hook A/B/C, gift-first, contrast, cluster, etc.] |
| Outcome | [accept / ignore / reply / bounce] |
| What worked | [specific thing that got a response or accept] |
| What didn't | [specific thing that got ignored] |
| Pattern spotted | [any cross-prospect observation — e.g., "brokers accept faster than investors"] |

### PRR Tracker Learnings
| Field | Value |
|---|---|
| City/Agency | [who the PRR was filed with] |
| Filing method | [email, portal, phone] |
| Response time | [days, or "pending"] |
| Outcome | [data received / pushback / redirect / bounce / denied] |
| What worked | [language, statute citation, specific fields requested] |
| What didn't | [too broad, wrong address, wrong department] |
| Portal/quirk discovered | [anything a future filing should know] |

### Product Agent Learnings
| Field | Value |
|---|---|
| Spec ID | [PROD##, INTAKE-###, or INFRA-###] |
| Spec gap found | [what the spec should have included but didn't] |
| CC deviation expected | [if CC will likely deviate, what and why] |
| Dependency missed | [anything this spec depends on that wasn't initially caught] |
| Fix for future specs | [one-line rule to prevent this gap next time] |

### QA Triage Learnings
| Field | Value |
|---|---|
| Issue type | [data / UI / functionality / trust] |
| Root cause category | [stale CSV, missing enrichment, mobile layout, config drift, etc.] |
| CC fix complexity | [quick < 30 min / medium 1-2 hr / needs spec] |
| Pattern match | [is this a repeat of a prior issue type?] |
| Prevention rule | [what automated check would catch this before it ships?] |

*Only include the table for YOUR agent type. Omit the others.*

### Strategy Agent Learnings (Strategy Chat only)
| Field | Value |
|---|---|
| Decision made | [what was decided — one line] |
| Alternatives considered | [what other options were on the table] |
| Why this option | [the reasoning — not just "it's better" but the specific logic] |
| Assumptions this rests on | [what has to remain true for this decision to hold] |
| Revisit trigger | [what would make you reconsider — specific, measurable if possible] |
| What this obsoletes | [prior decisions, approaches, or queued items this replaces] |

*Strategy learnings are appended to `ds-strategy-learnings.md` during session close.*

---

## Specs Ready to Send to CC

| Spec | File | Status |
|---|---|---|
| [spec name] | [filename] | [Ready / Needs review / Blocked by X] |

---

*[Agent Name] Handoff | [Date] | Session Close*
*Key outcome: [One sentence summary]*
```

---

## Section Guide — What Goes Where

**"What Happened"** — Describe outcomes, not process. "Tampa build spec written" not "We discussed Tampa and decided to write a spec." Include data: row counts, parcel counts, pass/fail results, file names.

**"Cascade Updates"** — This is the most important section. Every change must name the specific file AND describe the specific edit. "Update PIPELINE" is useless. "DS-CITY-PIPELINE.md — Move Tampa from 'In Progress' to 'Pipeline-Ready — both legs + spec'" is useful. If a change affects multiple files, list it under each file.

**"Open Decisions"** — Only decisions Jeff hasn't made yet. If Jeff decided during the session, it goes in "What Happened" and the cascade reflects the decision. Don't re-surface resolved items.

**"Session Learnings"** — What the agent observed that would make the next session better. Use your agent type's table format. This is how agents build memory across sessions. If the session was routine with nothing new learned, write "Clean session — no new learnings" and move on.

**"Files Produced"** — Every file the session output. Include destination (project knowledge, CC, outputs). Strategy Agent uses this to verify nothing is missed.

**"Specs Ready"** — Separate from files produced. These are CC-bound and may need Jeff's send approval.

---

## Sections to Omit When Empty

Don't include empty sections. If no scoring changes happened, don't write "DS-SCORING-REGISTRY.md — No change." Just omit it. The Strategy Agent's reconciliation protocol checks for pending items independently — it doesn't need a "no change" placeholder.

Exception: If a file SHOULD have been updated but intentionally wasn't (e.g., "Registry not updated until deploy confirms"), include it with the reason.

---

## Standing Rule for Agents

Added to every agent skill file in the fleet:

```
**Session Close Handoff:** Every session that produces changes must close with a
handoff document using the template in `handoff-template.md`. The handoff maps
all changes to specific project files for Strategy Chat reconciliation. A session
without a handoff means changes may be lost.
```

---

## What the Strategy Agent Does With This

1. Reads each handoff sequentially
2. Extracts the Cascade Updates section — this is the instruction set
3. Cross-checks against pending items already in TODO and project knowledge
4. Extracts Session Learnings — appends to the agent's learnings file (e.g., `ds-outreach-drafter-learnings.md`) in project knowledge
5. Produces updated project files with changes from all handoffs merged
6. Presents summary of what changed and what wasn't touched
7. At Strategy session close — captures Strategy learnings into `ds-strategy-learnings.md`

The better the handoff, the less interpretation required. Interpretation is where mistakes happen.

---

*Agent Handoff Template | Adapted from DeltaScanner's production fleet*
*"Tell the Strategy Agent exactly where things go. Don't make it guess."*
