# The Layer Nobody Designs For

### A model comparison surfaced four calibration misses in 36 hours — and one architectural layer that doesn't appear in any diagram I've seen | April 26, 2026

On April 25 I ran the same agent skill on Claude 4.6 and 4.7. Same skill file, same inputs, same protocol. I scored both runs against a six-dimension rubric — extraction completeness, ROI calibration, cross-referencing, synthesis, compile accuracy, signal-to-noise. **4.6 scored 49/60. 4.7 scored 55.5/60.**

I switched back to 4.6 anyway. The rubric measured what the skill produced. It didn't measure what the operator paid to get it.

## The 36-hour pattern

4.7 was more verbose. It presented options without recommending. It speculated where 4.6 would have acted. Day-of, I built a standalone markdown file encoding my working style — response length, directness, speculation discipline, decision tempo. The Operator Profile, v1.

Over the next 36 hours, three more calibration misses surfaced — under-verifying before pushing back, adding parallel curation layers when canonical sources existed, producing specs without the execution metadata needed to run them. Each became a new discipline section: Verification, Architecture, Spec Handoff. Profile bumped v1 → v4 in a single day.

The pattern wasn't "model bad, I caught it." Every miscalibration surfaced an expectation about my working style I'd never made explicit, because I'd never had to. The implicit calibration that had been carried in conversation reset to zero on a model upgrade. I was suddenly a stranger to the agent.

## What this reveals

**The interaction layer is invisible until it breaks.** Every agent-system framework I've seen starts at "how the agent works" and skips "how the operator calibrates the agent to themselves." The Operator Profile doesn't appear in any architecture diagram I've encountered. It exists in mine because four misalignments forced me to make implicit expectations explicit.

**Skill design beats model upgrade.** A different miss happened on both models — neither flagged that an enrichment data source was a complementary layer, not a competitor. The fix was a composition check baked into the skill file. The improvement worked on every model. Model intelligence is the commodity; skill design is the moat.

## Honest limit

The portability architecture has four layers (Operator Profile / Skill Logic / Model Adapter / Benchmark Results). Layers 1–2 are in production; Layer 4 — the standardized behavior battery — is partially built. The comparison rubric ran once. It isn't yet repeatable across model releases.

---

**Long-form available on request.** Walks the rubric, the four misses, the model-independent miss, and the portability architecture in depth.

*Operator Profile: [`examples/operator-profile.md`](../examples/operator-profile.md)*
