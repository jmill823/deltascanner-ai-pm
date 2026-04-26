# AI-PM Intelligence Brief

### Compiled from `jmill823/intel` | Last updated: April 24, 2026

**Purpose:** Living reference for the AI Product Management positioning that informs the case study in this repo. Compounded from thirteen batches of intelligence extraction across public talks, profiles, and analyses. Auto-regenerated each compile cycle from the source intel repo.

**Scope:** The principles below are conclusions derived from multiple insights across batches. Each principle cites its evidence chain. Insight numbers (`#147`, `#170`, etc.) reference internal artifact IDs in the live intel stream and are kept as a credibility signal rather than redacted — the brief itself is an instance of the institutional knowledge layer the case study describes.

---

## Executive Summary

Three convergent signals across the intel stream:

**1. The hiring bar has shifted from resume logos to system artifacts.** Interviewers in this segment no longer ask "walk me through something you shipped five years ago." They ask "what tools do you use?" and "show me the learner.md." The portfolio that wins is a live, versioned, compounding system — not a slide deck.

**2. The role is converging on fleet management, not feature management.** Multiple independent sources describe the same future: the human designs the system, curates the agents, verifies outputs, and feeds corrections back into the loop. This validates the curator-not-router thesis articulated in the case study's Layer 2.

**3. The compensation band is real and demand is accelerating.** Senior AI PM compensation at frontier labs sits in the $500K–1M+ range. Companies will continue restructuring toward AI-native builders. The premium is for designing systems that improve from operator judgment, not for using AI tooling.

---

## Key Principles (Compounded)

### P1: The two hiring differentiators are skill version history and the learner loop.

The two questions that separate builders from resume-updaters: *"Walk me through a skill you've built, and the evolution of the checklist over time"* and *"Show me the learner.md for one of your agents."* This repo already demonstrates both: every project file in the source system uses a versioned numeric suffix convention; the [`docs/solutions/`](./examples/solution-files/) directory IS the learner.md the question asks about. The case study doesn't invent these artifacts — it names what the system was already producing.

**Evidence:** Mahesh Yadav profile (#147), Garry Tan on skillification (#158), the source operator's Team OS positioning (#134-135).

### P2: A multi-agent product converges on a five-layer architecture.

The five-layer stack (#170) is the skeleton most agentic systems converge toward: interaction → orchestration → specialized agents → data → model API. The case study walks each layer with its production implementation in DeltaScanner. Tradeoff calls within each layer are the differentiator in technical interviews.

| Layer | Name | Production implementation |
|---|---|---|
| 1 | Interaction | Operator Profile + `deltascanner.com` |
| 2 | Orchestration | Strategy chat — routing, gates, handoff contracts |
| 3 | Specialized Agents | Eight agents with versioned skill files |
| 4 | Data | Compounding intel repo + `docs/solutions/` |
| 5 | Model API | Multi-provider routing (Claude, Claude Code, Perplexity, ChatGPT/Codex) — by design, not by convenience |

**Evidence:** Five-layer architecture analysis (#170), the source operator's Team OS positioning (#134-135).

### P3: "Modernity" is the new credential. The live system is the resume.

Recent commentary from senior PMs in the segment names this directly: candidates who spent extended time at established companies working in pre-agentic patterns come out with the world having moved on. The new interview opener is "put yourself in a scenario — what tools do you use?" The right answer is not a list — it's a walkthrough of a live, running system, with the architecture, the compile cycles, the agent fleet, and the feedback loops shown end-to-end.

**Evidence:** Nikhyl on the modernity signal (#137), Cat Wu on taste over title (#171), Mahesh Yadav on system-design over tool-use (#147).

### P4: The curator-not-router thesis is validated by Anthropic's own product lead.

Cat Wu (Claude Code product lead) framed the future of work as managing fleets of AI agents rather than doing the work yourself (#171). The human role shifts to knowing which tasks to investigate, verifying outputs, and giving feedback that makes the system better. This is the case study's Layer 2 thesis, validated independently from a senior product role at the model provider.

**Evidence:** Cat Wu on fleet management (#171), source operator's Team OS pitch (#134), Nikhyl on the PM renaissance (#136).

### P5: The institutional knowledge layer is the moat, not the model.

The thesis: models swap every six months; the shared knowledge layer (skills, commands, automations, version-controlled and operationally read by the agents) persists. That layer is the moat. This is independently supported by Andrej Karpathy's published thinking on the durable knowledge base (#55-56), Garry Tan's resolvers and skillification framings (#117, #158), and Cat Wu on internal tool building (#171). The DS intel repo is itself a production instance of this thesis — over 170 numbered insights, thirteen batches, compounding.

**Evidence:** Source operator's Team OS thesis (#135), Karpathy on knowledge bases (#55-56), Tan resolvers (#117), Tan skillification (#158), Cat Wu on building internal tools (#171).

### P6: Every model release should trigger a scaffolding audit.

Anthropic actively strips features that weaker models needed when stronger models ship — Claude Code's to-do list was scaffolding for models that couldn't track their own work, removed when Opus 4 handled it natively (#171). Applied to a fleet operator: every agent skill file should be audited on model upgrade for rules that are now handled natively, and for rules that just broke because the model's defaults shifted. The case study's Layer 1 (Operator Profile) is the artifact that emerged from this audit pattern in production.

**Evidence:** Cat Wu on scaffolding (#171), recent migration analyses on the 4.7 release (#160), Garry Tan's skillification framework first step (#158).

---

## Auto-Generation

This brief is regenerated each compile cycle by reading all `raw/*.md` files in the source intel repo, filtering by `projects` containing `ai-pm`, sorting by ROI then date, and producing the brief in markdown. The generation script is documented in the source repo. The brief is therefore not a static artifact — it's an instance of the live institutional knowledge layer that Principle P5 describes. Reading this brief is reading the layer the case study claims is the moat.

---

*AI-PM Intelligence Brief | `briefs/ai-pm-brief.md` | Auto-update target: every compile cycle*
*The brief is itself the artifact the principles describe. P5 in published form.*
