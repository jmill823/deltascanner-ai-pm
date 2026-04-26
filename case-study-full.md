# DeltaScanner as an AI Product Management Case Study

Last week a model upgrade broke my agent system.

Not visibly — the agents ran, the outputs came back, the deploy reports showed green. But the calibration was off. Responses got verbose where they used to be tight. The system started presenting equal-weight options instead of recommending. Speculation arrived undecorated, presented at the same confidence as verified work. The skill files hadn't changed. The data hadn't changed. The model had changed, and a layer of the system that I'd never named — the layer between the operator and the agents, the one that calibrates communication style and decision posture — was now misaligned.

So I built that layer explicitly. An [Operator Profile](./examples/operator-profile.md). A model-agnostic file that encodes how I work, what I expect, what wastes my time. Any agent in the fleet reads it and calibrates accordingly. The next model upgrade won't break the system the same way, because the layer that was implicit before is now a file I version, audit, and shim per model release.

That's the thesis of this case study. **The architecture wasn't designed. It was discovered through operating.** Five layers, ninety days, one solo operator. Every layer in the stack exists because something specific broke and I encoded the fix. The system isn't what I planned to build — it's what I learned a multi-agent system needs by running one in production.

---

## What this case study covers

DeltaScanner is a distressed property intelligence platform. Live across thirteen U.S. markets and scaling, ~223,000 scored parcels, eight agents in production, one operator. The product thesis is workflow compression — turn validated parcel-level distress signal into operator decisions in seconds. That's the user-facing claim, and it's defensible.

The interesting claim isn't the product. It's the operating model that produced it. I'm a non-technical solo founder. I do not write build code. The agents write, build, deploy, and surface work for review. I approve, reject, or defer at scheduled gates. Between gates, my inbox is empty by design. That's not a workflow choice — it's a structural property the system is engineered to maintain.

This case study walks the architecture top-down: Layer 1 (the operator profile and product surface), Layer 2 (orchestration), Layer 3 (specialized agents), Layer 4 (compounding data), Layer 5 (model routing). For each layer, the same shape: what it does, the failure that created it, the evidence that it's working. At the end, the limitations I can't defend a stronger claim against, the experiments running now that would change the thesis, and the metrics I'm measuring forward.

---

## Layer 1 — Interaction

### What it is

The interface between the human and the system. Two artifacts live here. The product surface — `deltascanner.com`, where buyers see scored parcels and filter on distress signal. And the Operator Profile — the calibration file that tells every agent in the fleet how I prefer to work. Same layer, two faces: one for buyers, one for me.

### The origin failure

The Operator Profile didn't exist three weeks ago. The product surface did. What I was missing was the layer that calibrates *how* the agents communicate with me, independent of *what* they do. That gap stayed invisible until a model upgrade exposed it.

April 25, 2026: a routine intel-extraction batch produced output that was technically correct but operationally wrong. Verbose where I needed terse. Options presented without recommendations. Speculation undifferentiated from verified claims. The skill file specified the task; it did not specify the communication contract. When the underlying model's defaults shifted, the implicit calibration shifted with it. I'd been carrying the contract in my head — and in my running corrections inside chats — for months without noticing.

The fix was a file: `operator-profile-v1.md`. Verbosity ceiling, recommendation posture, speculation rules, decision tempo, format defaults, anti-patterns, and per-model adjustment notes. Model-agnostic by design — it states preferences, not prompts. Per-model shims live in a separate section that gets updated after each model release runs through a five-input behavior benchmark.

### Why this layer matters

Most architecture diagrams of agentic systems start at Layer 2 (orchestration) and go down to Layer 5 (model API). Layer 1 — what the operator does, how the operator talks to the system — gets treated as ambient. It isn't ambient. It's the layer most likely to break first when a model upgrades, and the layer least likely to be documented when it does. The Operator Profile makes that layer explicit and portable. If I switch model providers tomorrow, the file ports with me. The skill files port with me. What's per-model is the shim section, which is small and rebuildable.

The honest limit here: the Operator Profile is one operator's. It doesn't scale to a team without becoming a Team Profile, which would need to encode disagreements, role-specific defaults, and override hierarchy. That's a different problem from what I'm solving. What I have works for solo operation, which is the only case I can defend.

---

## Layer 2 — Orchestration

### What it is

The routing layer. Decides which agent gets which task, which gates apply, what handoff contract a piece of work follows from agent to agent. In DeltaScanner, this layer is operated by a Strategy chat — categorization, spec generation, reconciliation across parallel agent outputs, session-close handoff into versioned project files. The Strategy chat is also where I sit. I'm not a separate layer; I'm an actor inside Layer 2.

### The origin failure

This layer exists because I tried to operate without it and the system degraded twice. The first degradation was substrate miscalibration: in three separate sessions over one week, I proposed architectural directions that contradicted standing doctrine because I was reasoning from fresh context rather than bounding against existing decisions. The fix was four behavior rules — doctrine check before proposing direction, "undecided" requires doctrine verification, name operator-orchestrator-reviewer before designing systems, contradiction gate before shipping. Each one exists because of a specific dated incident.

The second degradation was a queue-backup failure mode. When the gate queue grew faster than I could review, I started approving to keep it moving. That's the failure mode that kills curator-mode if you don't build against it. The structural response was to make every handoff follow a [contract](./handoff-template.md) — build specs include reproduction steps, acceptance tests, post-deploy QA checks, scope boundary; miss any one and the spec is incomplete, I don't read it, I reject it. The contract is what makes gate review fast: scan a known structure for known fields in 30-60 seconds rather than reading prose and rebuilding context.

### How it actually works

Three things have to be true for batched review to work, and all three are running in DS now.

*Each agent has a lane and stays in it.* Strategy categorizes and specs but never writes build code. Claude Code writes code but never decides what to build. The Outreach Drafter drafts but never sends. The PRR Tracker logs but never adjudicates. If an agent drifts outside its lane, the handoff it produces looks wrong at the gate before anything ships downstream — that's how lane drift surfaces.

*Every handoff follows a contract.* QA tickets need category, steps, expected, actual. PRR rows need jurisdiction, statute, filed date, follow-up-due. Session-close handoffs need cascade updates mapped to specific project files, not generic "update PIPELINE" routing. The contract format is itself versioned — when the format changes, the version bumps, and downstream consumers know which schema to expect.

*Knowledge compounds across sessions.* Layer 4 covers this fully. For the orchestration picture, what matters is that the gate inherits every prior lesson without me holding it in my head between reviews.

### Trade-off and evidence

Real-time human-in-the-loop routing caps throughput at my personal availability — workable for a technical founder, fatal for a non-technical one. Batched review uncaps throughput but introduces real costs: up to 48 hours of latency between gates, and the queue-backup failure mode if preconditions weaken.

DS operated without formalized acceptance test standards through March 2026. Builds flowed faster, but rework rate was high and scoring defects leaked to production. The March 30 architecture audit — ad hoc, not standing — surfaced defects in 8 of 13 live city models: recency components still active after the recency ban, weight drift from documented values, architecture labels that didn't match actual scoring behavior. None had been caught by the existing QA gate because the existing gate tested the display layer, not the scoring layer.

The structural response was four-part: a YAML-configured universal scorer replacing 13 per-city scripts, schema validation enforcing weight-sum constraints and blocking recency-named components in the weighted composite, [acceptance test standards](./examples/cc-spec-standards.md) requiring testable assertions written before any build code, and a Codex external review gate on architecture-touching changes. The batching rhythm didn't change. The precondition floor rose.

| Metric | Pre-April 3 | Post-April 3 |
|---|---|---|
| First-pass build acceptance | 4 of 13 (31%) | 6 of 7 (86%) |
| Scoring defects in production | 8 (revealed retroactively by audit) | 1 (Dallas zip, caught by post-deploy QA) |
| QA batch triage (N=26, early batches) | ~60% approved to build, ~31% routed for spec, ~4% rejected, ~4% caught as false-positive | Forward logging began April 22 |

Hialeah, Tampa, West Palm Beach, Los Angeles, and additional cities are in flight and will close within six weeks. The methodology is tested on a growing build set, not a frozen retrospective.

---

## Layer 3 — Specialized Agents

### What it is

The agents themselves — eight in production, each with a single responsibility, a versioned skill file, and a defined handoff format. The fleet covers PRR research and tracking (PRR Tracker), data acquisition discovery (Data Discovery), build execution (Claude Code via spec routing), product intake and feature spec generation (Product Agent), site QA and triage (QA Triage Agent), prospect outreach drafting (Outreach Drafter), buyer-facing deliverable QA (Deliverable QA), and continuous intel extraction from cross-source reading (CC Intel Queue).

### The origin failure

The fleet didn't start as eight agents. It started as one chat with me trying to do everything in it. The agents accreted as failure modes appeared: outreach quality drifted because I was drafting under time pressure (Outreach Drafter), QA observations bled into product decisions because there was no triage layer (QA Triage), product intake skipped dependency checks because I was excited about ideas (Product Agent), reading list backed up because intel extraction got deprioritized whenever any other work appeared (CC Intel Queue). Each agent exists because a specific class of work needed its own lane and its own quality standard.

### How agent boundaries hold

Three properties keep agent lanes from blurring. First, each agent has a skill file with explicit triggers, scope, and exclusions — what the agent does, what it does NOT do, and which other agent owns the adjacent work. Second, every agent's output is structurally distinct: a PRR row looks nothing like a feature spec, which looks nothing like a draft DM. The format itself reveals lane drift. Third, the Strategy chat is the only chat that updates project files; agent outputs flow back through Strategy for canonicalization. That centralization is what prevents conflicting state across parallel agent work.

The honest limit: lane definition is informal in the sense that there's no enforcement layer that prevents an agent from drafting outside its scope. The enforcement is the handoff contract — a CC build that wasn't requested by Strategy doesn't have a spec, and a deploy without a spec is rejected at the gate. Indirect enforcement, but it's what's run for four months without lane breaches that I'd characterize as material.

---

## Layer 4 — Data

### PM framing up front

I designed the protocol below, specified its acceptance criteria, and enforced its adoption across the agent fleet. Claude Code implemented it. That split is the job — frontier-lab AI PM work is defining what "done" looks like for a system you won't write the code for, then making sure the implementation matches the specification. The protocol is load-bearing for DS's current operating tempo. It's also the clearest example I have of specifying a technical system at the level of detail a hiring manager would expect.

### The origin failure

Every Claude Code session started from zero. A build on city #12 could make a mistake that city #3's build had already solved. The repo accumulated commits but not lessons. My earlier attempt at knowledge persistence — a manually maintained markdown file called `CC-LESSONS-LEARNED.md` — failed for the reason these files always fail: I was the bottleneck on writing it, which meant it only updated when I remembered to update it, which meant it was stale by the time anyone needed it.

The underlying design gap: the system had decision gates, execution loops, and evaluation checks, but no cross-session memory. Without that, every new build paid the cost of every old build's discoveries.

### The protocol

The Knowledge Compounding Protocol (INFRA-001), shipped April 5, 2026, moved the knowledge store out of my hands and into the system's operating loop. Four elements define it.

*Write contract.* After every build, Claude Code writes a structured solution file to `docs/solutions/` — one file per distinct lesson, with a canonical schema: failure class, symptom, root cause, the rule that should prevent recurrence, the affected files. No free-form prose. 36 files currently seeded.

*Read contract.* Before every new build, Claude Code reads all prior solution files and summarizes the relevant ones in its plan. The read step is a required gate; a build that skips it fails acceptance.

*Contradiction handling.* When a new solution contradicts an existing one, Claude Code surfaces the contradiction as a BLOCKING flag in the deploy report rather than silently overwriting. I adjudicate. The old rule is either revised or retired with the reasoning logged — no silent drift.

*Acceptance tests.* Sixteen tests defined what correct protocol execution looked like: solution file schema validation, read-before-write ordering, contradiction detection, retrieval relevance. All sixteen passed on first implementation.

### The evidence

Three solution files demonstrate the protocol catching real regressions it was built to catch.

*[Hardcoded filenames](./examples/solution-files/hardcoded-filenames.md) (April 2).* Miami's enrichment silently broke when a rescore changed the output CSV filename. Root cause: `enrich_parcels.py` referenced a dated filename instead of using a dynamic finder. The solution file captured a rule — "enrichment scripts must dynamically locate the current scored CSV" — that has since prevented the same class of failure in three subsequent builds.

*[Forced sale file finder](./examples/solution-files/forced-sale-file-finder.md) (April 4).* During the PROD40 Chicago build, Claude Code's `find_latest_forced_sale()` function selected a file matching the naming pattern but not containing the expected columns. Rather than ship a silent data loss, Claude Code caught the gap, fixed it autonomously, and wrote a solution file: "file finders must verify column contents, not just filename match." This is the protocol working end-to-end — no human in the loop, no spec update required, the system generalized from a prior lesson.

*[Pipeline gate false positive](./examples/solution-files/pipeline-gate-false-positive.md) (April 3).* The PROD30 atomic pipeline flagged NYC as non-compliant because `score_recency` existed in the config. The field was metadata, unweighted. The resulting solution file tightened the rule: "pipeline validation checks the weighted composite, not field existence." Every subsequent compliance check uses the weighted version.

The solution file count grows as Hialeah, Tampa, West Palm Beach, and Los Angeles close. The protocol is tested on a growing corpus, not a closed one.

### What this protocol is not

It's not continual learning in the frontier-lab research sense. No gradient updates, no weight modification. It's structured retrieval over a curated document store, durable across sessions. The distinction matters because I won't oversell this as something it isn't — what it does is measurable and working, what it doesn't do is equally clear.

It's also not a general-purpose memory system. The solution files are narrow and domain-specific. They encode DS-specific build knowledge, not broad reasoning patterns. A frontier lab's equivalent would require far more sophisticated retrieval, consistency management, and scope. What DS has is the minimum viable version, sized for a single operator running an eight-agent fleet.

---

## Layer 5 — Model API

### The principle

A model that produces work can't objectively audit that work. This isn't a statement about any one model's quality; it's a structural property of systems that ask the same reasoning process to both generate and evaluate its own output. The architectural response is to separate the maker from the checker at the model level, not just the prompt level.

### The origin failure

April 1, 2026 — Codex Gate 2. Claude Code produced a handoff documenting four standing rules that referenced file paths that didn't exist in the deployed repo. The rules described an architecture (`city-config.json` plus 13 per-city scripts) that was internally consistent but factually wrong — Claude Code had built a cleaner architecture (YAML configs plus a universal scorer) and documented the spec rather than the implementation. Codex, auditing the repo independently, caught it. If the build had used Claude Code for both construction and review, the incorrect standing rules would have been enforced going forward. The same-model-as-its-own-auditor failure mode isn't hypothetical.

A second related incident, March-April: Claude Code's web search proved a poor fit for iterative property validation — each lookup pulled full page content into its context window, so by property #30 the session was dragging every prior result. A task the model was nominally capable of produced systematically worse results the longer the session ran. This is a model-as-tool fit question, not a capability question. The routing fix was Perplexity for interim validation and a queued script for production replacement.

### The routing

DS applies maker-checker separation across four models. Claude (this chat) handles strategy, synthesis, and the document layer — where judgment and cross-session continuity matter. Claude Code handles build execution, repo operations, and acceptance-test enforcement — where a different orchestration surface and bash access change what's feasible. Perplexity handles city and dataset discovery. The primitive there is search-with-citation-at-synthesis-time — every claim arrives already sourced, rather than produced as prose and optionally cited after the fact. ChatGPT (and Codex, for repo audits) handles adversarial review — where structural independence from the builder is the requirement.

The separation is built into the agent fleet, not bolted on. Claude Code builds; ChatGPT or Codex reviews. Claude drafts outreach; ChatGPT pressure-tests it. Claude writes buyer deliverables; a skeptic pass from a different model family catches what the builder model missed.

### Honest limit

The routing is manual. I decide which work goes to which model, which skeptic pass runs when, which handoff schema applies to a novel artifact. There's no routing agent, no enforcement that skeptic review must run before certain deploy classes. Scaling past a single operator would require either a stricter protocol (skeptic review as a required gate for class X of build, enforced programmatically) or a routing agent that makes selections itself. Neither exists yet.

A separate experiment is running to test whether the skeptic role specifically requires a different model family, or whether the separation principle could be satisfied within a single family using different prompts or contexts. If cross-family review and same-family review catch defects at equivalent rates, the routing logic simplifies and the fleet shrinks. If they don't, the four-model architecture is justified on structural grounds.

---

## What this evidence does not prove

Four structural changes co-occurred in the first week of April: architecture audit, universal scorer refactor, acceptance test standards, Codex external review gate. I can't causally isolate which produced most of the acceptance-rate gain. What I can claim is that structural tightening and rate improvement move together, which is consistent with the broader thesis that coordination cost belongs in handoff contracts rather than the operator's head.

The Goodhart risk on gate metrics is real and is mitigated — not eliminated — by complexity-weighted build counts, retroactive defect traceback on 30-day post-deploy bugs, and reject-reason categorization. Faith that agents won't optimize for a measured gate is not a mitigation. The honest read is that my attention is the enforcement layer for the Goodhart safeguards. If I stopped applying them, the gate metrics would drift toward looking good rather than being good, and I might not notice. Scaling this system past me needs to move that enforcement into the protocol itself, which I haven't specified yet.

The "designed by operating" thesis carries a related risk: it's a description of the path that produced this system, not a methodology that generalizes to every multi-agent system. A system with different domain economics, different operator constraints, or different downstream stakes might require explicit upfront architecture. What I can claim is that for one solo operator running this domain at this scale, the architecture that emerged from operation is the architecture that holds up under operation. Whether it would have been faster to design the same five layers upfront is unknowable; what's testable is whether the layers I have hold as the system scales.

---

## What I'm measuring next

Three things between now and mid-June.

One, whether the post-standards first-pass build acceptance rate holds at ~86% as the sample expands from 7 to 12+ builds. A material drop tells me one of two things: either I've reached the edge of the current precondition set, or the early sample was favorable because the audit had already surfaced the easy defect classes.

Two, the result of a running experiment on cross-family vs. same-family skeptic review. If the catch rates are equivalent, the model-routing logic in Layer 5 simplifies. If they're not, the four-model architecture is justified on structural grounds and the case for formalizing routing strengthens.

Three, the false-positive rate on the QA triage gate. The March sample caught one false positive across 26 items (~4%) — meaning the gate correctly rejected work that looked like a bug but wasn't. That's a feature of the system working. A rate that drifts too high would mean the QA Triage agent is miscategorizing; a rate that collapses to zero would mean the agent is approving items that shouldn't reach me. Either failure mode is visible in the metric.

A fourth experiment is the most direct test of the Layer 1 / Operator Profile claim. Extending the Knowledge Compounding Protocol from Claude Code to the Outreach Drafter gives me a measurable surface — DM acceptance rate, reply rate, conversation conversion — to test whether structured cross-session memory works for an agent producing communication artifacts, not just build artifacts. If the protocol generalizes, Layer 4 is mechanism, not Claude-Code-specific accident. If it doesn't, the protocol's scope narrows and the case study revises in that direction.

---

## What would change my mind about the thesis

The "designed by operating" thesis rests on a claim I can't fully test alone: that the coordination cost held by agents and handoff documents scales sub-linearly with fleet size, while the coordination cost held in an operator's head scales linearly. If I added three more agents and discovered gate review time doubled rather than grew marginally, the thesis would need revision — the preconditions wouldn't be holding at larger scale. I don't expect this, but I haven't proven it. A year of operation at double the current agent count is the experiment that would settle it.

The first test lands sooner. The Outreach Drafter extension is the next experiment. If the protocol generalizes from build execution to outreach, the mechanism the case study claims is real. If it doesn't, the protocol is Claude Code-specific and the thesis needs narrower framing — closer to "designed by operating Claude Code" than "designed by operating a multi-agent system."

The case study is what I learned running the system I have now. What I haven't yet learned is what this system looks like at scale, under a team, or under pressure from defect classes the audit didn't reveal. That work is in front of me. The architecture diagram is finished; the architecture is not.

---

*Word count: ~3,450 words. Authored April 2026. The system described in this case study is in operation, not in retrospect.*
