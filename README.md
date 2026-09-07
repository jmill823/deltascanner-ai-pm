# AI-PM proof, in 3 minutes

I'm a non-technical, AI-native PM — former Head of Product / COO at Deepblocks (0→100+ B2B clients). What follows is solo-built, self-operated, and non-commercial. Five lines, each linked to the proof.

1. **I operate a multi-agent system.** An 8-agent fleet on Claude that authors and validates a deterministic decision engine — live docket monitoring across two Florida counties, hand-verified weekly, ~2,200 parcels under watch. → [Deltascanner](https://deltascanner.com)
2. **The failure I kept hitting:** AI systems overclaim and drift — a confident, fluent answer instead of "I don't know," and behavior that drifts silently when the model changes, no error thrown.
3. **Structural Mistrust is the discipline that catches it.** Confidence computed, not asserted; failures marked, not laundered; high-stakes judgments frozen until they survive disagreement. → [the brief](https://jmill823.github.io/deltascanner-ai-pm/structural-mistrust-brief.html) · [a kill call, live](https://jmill823.github.io/deltascanner-ai-pm/pre-registered-kill.html)
4. **Ledger → Launchpad is the execution surface.** The operator loop made interactive — a compounding ledger compressed into discrete, provenance-carrying actions. → [live demo](https://jmill823.github.io/deltascanner-ai-pm/intel-launchpad.html) · [ported to a second domain](https://jmill823.github.io/deltascanner-ai-pm/operating-kernel.html)
5. **The honest limit:** the outcome loop is instrumented but hasn't accumulated enough reality to score its predictions against yet. I won't claim otherwise.

---

**How it was built:** I don't write code — an 8-agent fleet does, in five layers, each one added because something specific broke (model upgrade broke calibration → Operator Profile; handoff failures → per-agent skill files; dark cross-session memory → Knowledge Compounding Protocol; same-model-as-its-own-auditor → maker-checker across four model families). Full case study, architecture diagram, ADRs, and the append-only catch ledger are indexed in [`CONTENTS.md`](./CONTENTS.md). The AI is in the build and review loop; the product's scoring and kill decisions have no model in the runtime.

*Published April 2026 · updated September 2026 · Jeff Millett · [Deltascanner](https://deltascanner.com) — solo-built, non-commercial · also live: [Tilt](https://playtilt.io), commissioner-first office-pool platform, and FFL-Intel, a fantasy-football pricing engine served as an MCP tool.*
