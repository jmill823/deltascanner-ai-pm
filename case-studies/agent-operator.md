# Case Study: Agent Operator Discipline at Scale

**One four-hour session. Three repositories hardened. Five PRs merged. One wrong locked decision caught and reversed. Zero work lost.**

---

Recently, I ran a single strategy session that retrofitted three private repositories — an agent-orchestrator framework I maintain plus two consuming projects (Tilt and a new instrument I am building) — onto a unified architecture. These were the first of other projects I hoped to unify using a similar OS.

The session's deliverables — merged pull requests, cleaned worktrees, a new locked canonical decision — are the visible output. The case study is about what made them possible: a tight set of operating disciplines I enforce when working through Claude Code (CC) and Claude.ai (c/ai) strategy chats as agents under my direction.

I'm calling this Agent Operator work. It's distinct from agent system *design* and from traditional product management in its current form. It's the operational craft of running multi-agent systems through real, structural work without losing artifacts, without letting drift compound, and without confusing pattern-completion for judgment.

---

## The disciplines that held

These aren't post-hoc rationalizations. They are the operating principles I kept enforcing across the session, which the session log shows in real time. Each one prevented a specific class of failure that would have cost the work.

### 1. Pre-flight before any structural work

Every spec started with a read-only pre-flight check that verified actual state on disk against expected state. No work was authorized until pre-flight passed.

This caught: a missing GitHub remote on the instrument repo; five dirty worktrees that would have lost real work on blanket force-removal; and a default-branch inversion (`master` instead of `main`).

If I had skipped pre-flight I would have lost spec outputs from earlier API audits. At worst, the entire retrofit foundation.

### 2. Inspect before destruction

When CC reported five worktrees refusing safe removal due to dirty state, the next spec was *read-only inspection of all five*, not authorization to force-remove. The inspection report showed three of five contained load-bearing content:

- `KERNEL-AUDIT.md` (1,477 lines, sister to a recently-committed framework audit)
- Spec execution outputs with verdicts that drove downstream decisions
- A 1,205-line prereq commit that turned out to be the foundation of the retrofit, not drift

A blanket removal would have destroyed all three. The discipline cost ~10 minutes of inspection time and saved hours of recovery work plus an unrecoverable foundation commit.

### 3. Read content, not labels

The original session handoff called the prereq commit's contents "drift artifacts" and recommended discarding them. The artifacts had names matching what other handoff documents flagged as drift: `skills/`, `activation-reference.md`, `HANDOFF-TEMPLATE.md`. Pattern-matching on filenames produced the recommendation.

Before authorizing destruction, I gated on a read-only spec that surfaced the actual content of the rewritten README and a sample skill file. The README articulated a coherent multi-agent architecture (direct-to-git handoffs, mode-locked agents) that explicitly improved on the framework's existing patterns. The skill file showed substantive product framing ("the data layer is the product"), not boilerplate.

The discard call was reversed. The prereq commit became the retrofit foundation. A new locked decision was written to formalize the supersession of the original that had set "lightweight, single-agent" scope incorrectly earlier.

**This is the moment the case study turns on.** The system was about to discard load-bearing work because pattern-matching pulled toward a coherent-sounding answer. The discipline was: don't authorize destruction without reading the actual content. That single gate prevented the wrong outcome and produced a new mandate, which is now load-bearing for everything that comes next.

### 4. Salvage before destruction

When a worktree needed to be removed but contained content I wanted to preserve, the spec copied content to a neutral temp directory *before* authorizing the destructive operation. The temp directory was consumed in a follow-up commit after the destructive operation completed.

This pattern decouples salvage from destruction. If something goes wrong with the destructive step, the salvage is already complete. If something goes wrong with the salvage step, no destruction has happened yet. Neither operation can corrupt the other.

### 5. Canonical-first reading at every decision point

Locked decisions are inlined verbatim at the top of project instructions as Layer 0. Before producing any structural recommendation, the discipline is to *consult* canonical, not just *have access to* canonical.

The session demonstrated this in the inverse: when canonical-first wasn't being followed, pattern-completion bypassed it. The original handoff had access to the prereq commit's content but didn't consult it before labeling it drift. The fix wasn't "be more careful" — it was a structural protocol where every spec touching structural questions has a read-the-actual-content step *before* the recommendation.

---

## Three excerpts from the session (sanitized)

These are receipts. Each illustrates a specific discipline catching a specific failure mode in real time.

### Excerpt 1 — CC self-correcting a miscount mid-output

```
> Total removed cleanly: 5 of 8 worktrees + 5 of 8 claude/* branches

Wait — that's 5 dirty worktrees. Let me recount: 1 in agent-
orchestrator-v1, 1 in tilt, 3 in instrument = 5 dirty
worktrees, 3 cleanly removed. Earlier "5 of 8" was wrong.
Correct count: 3 of 8 removed cleanly, 5 refused.
```

*Discipline shown: agent self-corrects without prompting. The operator's job is to design specs that make this kind of self-check natural, not exceptional.*

### Excerpt 2 — pre-flight catching the missing remote

```
## instrument
- Local path: C:\Users\mille\projects\instrument
- Remote: ⚠ NONE — no `origin` configured, `gh` reports
  "no git remotes found"
- Default branch (remote): N/A (no remote)
- Current branch (local): master

## Readiness assessment
- Ready for retrofit: N. Hard blockers:
  1. No git remote — repo is local-only, so any retrofit work
     has nothing to push to and no protection rules can be
     applied.
```

*Discipline shown: pre-flight surfaced a hidden risk that would have made the entire retrofit unrecoverable on disk failure. The remote was created in the next spec; the no-remote risk window closed about an hour after surfacing.*

### Excerpt 3 — operator override after canonical-first read

```
The README is genuinely good — and that changes the recommendation.

What I'm seeing: that README is not drift. It's a coherent, well-
thought-out scaffolding document that articulates an agent-
orchestrator pattern parallel to (but distinct from) what tilt
ended up with...

This is the instrument drift mechanism in reverse: pattern-completion
("these look like drift artifacts") bypassed the available
information ("read the actual files first").
```

*Discipline shown: operator gated on a read-only content surface before authorizing destruction. The previous recommendation (discard) was reversed once the actual content was visible. The decision that resulted in a mandate is now load-bearing for the entire instrument project.*

---

## What this signals

The through-line is: **I treat agents as I would treat a junior engineer whose memory resets between tasks.** Specifically:

1. **I write specs with verifiable success criteria, not vibes.** Every CC spec ended with explicit verification steps. CC's job was execution; mine was gating.

2. **I assume agents pattern-complete unless structurally prevented.** Canonical-first as Layer 0, content-read before labels, pre-flight before recommendations — these are protocols against pattern-completion, not against the agent.

3. **I treat every recommendation as draft until verified.** Including my own. The session log includes me reversing my own recommendations twice as new information surfaced. The discipline isn't "always be right"; it's "be willing to update fast."

4. **I make decisions visible and durable.** The new mandate isn't in a chat log. It's in a canonical decisions file, referenced by every future strategy chat that opens that project. The decision survives the context window because the structure makes it survive.

5. **I plan for handoff before I plan for completion.** The session ended with two handoff documents written for two distinct next-session goals, not one. Conflating them would have forced future-me to parse what applied to what.

---

## Background context

I'm Jeff Millett — non-technical solo founder operating Deltascanner (an 8-agent fleet on Claude scoring ~223,000 property parcels across 13 US markets for distressed property intelligence), building Tilt, and other fun tools, while conducting a structured AI-PM job search. Prior: product leadership at Deepblocks (B2B SaaS PropTech, B2C-to-B2B pivot, scaled 0 to 100+ clients in 10 months). M.Arch from Columbia. Computational art practice that informs my systems-thinking discipline.

The "Agent Operator" positioning is the actual work I do daily — running agents through structural problems, catching their drift before it compounds, producing artifacts that survive context windows.

---

## Receipts

- Live PRs from the session (5 merged) — private repos, reviewer access on request
- Session handoff documents (2) — available on request
- Locked canonical decisions, including the supersession discussed above — available with reviewer access
- Full sanitized session log (~30k words) — available on request
- Deltascanner public artifacts — github.com/jmill823/deltascanner-ai-pm
- Deltascanner site — deltascanner.com

Reviewer access requests and direct contact: **millett.jeffrey@gmail.com**

---

*Case study captured 2026-05-08. The session's actual log is available in sanitized form on request. The excerpts above are verbatim from the session, with no retroactive editing.*
