# Operator Profile (one-pager)

### The Layer 1 calibration file. Model-agnostic. Ports across the fleet.

A standalone markdown file encoding how the operator works — response length, directness, speculation discipline, decision tempo, verification posture, architecture discipline, spec-handoff metadata. Read by every agent at session start. Lives in the agent OS repo, git-versioned, model-version-shimmed where needed.

## Why this file exists

A model upgrade broke implicit calibration that had been running fine for months. The skill files specified the task; they did not specify the communication contract. Verbose where I needed terse. Options presented without recommendations. Speculation undifferentiated from verified claims. The fix was the file.

Three subsequent failure modes surfaced over the next 36 hours — under-verifying before pushing back, adding parallel curation layers when canonical sources existed, producing specs without execution metadata. Each became a new discipline section. The profile grew v1 → v4 in a single day. The version stack is itself the evidence — three named failure modes, three discipline sections, all encoded at moment of catch.

## What the structure encodes

Discipline sections per failure class. An anti-patterns catalog ranked by frequency. Per-model shims scoped tightly so the rest of the file stays portable. A changelog that names the triggering incident for every section added.

When a model changes, the profile carries calibration forward. The shim section compensates for known behavioral differences without rewriting the profile itself. The cost of a model migration drops from "rebuild the agent" to "run the benchmark and write a shim."

## Honest limit

Generalization is the open question. The profile encodes one operator's working style. Whether the *structure* (calibration file, discipline sections, per-model shims, changelog of triggering incidents) generalizes — or whether what works for me is mine alone — is unmeasured. The portability architecture is in production; the generalizability test isn't.

---

*Full file (~2,400 words, sanitized): [`examples/operator-profile.md`](../examples/operator-profile.md). Walks every section, every rule, every anti-pattern, the per-model shim structure, and the v1 → v4 changelog with named triggering incidents.*
