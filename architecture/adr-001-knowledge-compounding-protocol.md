# ADR-001: Knowledge Compounding Protocol

**Status:** Accepted — April 5, 2026
**Layer:** 4 (Data)
**Author:** Operator (specification); build agent (implementation)

---

## Context

Every build-agent session started from zero. A build on city #12 could make a mistake that city #3's build had already solved. The repo accumulated commits but not lessons. The earlier attempt at knowledge persistence — a manually maintained `CC-LESSONS-LEARNED.md` — failed for the reason these files always fail: the operator was the bottleneck on writing to it, which meant it only updated when remembered, which meant it was stale by the time anyone needed it.

The structural gap: the system had decision gates, execution loops, and evaluation checks, but no cross-session memory. Without that, every new build paid the cost of every old build's discoveries.

## Decision

Move the knowledge store out of the operator's hands and into the system's operating loop. Four elements define the protocol:

1. **Write contract.** After every build, the build agent writes a structured solution file to `docs/solutions/` — one file per distinct lesson, canonical schema (failure class, symptom, root cause, prevention rule, affected files). No free-form prose.
2. **Read contract.** Before every new build, the build agent reads all prior solution files and summarizes the relevant ones in its plan. The read step is a required gate; a build that skips it fails acceptance.
3. **Contradiction handling.** When a new solution contradicts an existing one, the build agent surfaces the contradiction as a BLOCKING flag in the deploy report rather than silently overwriting. The operator adjudicates. The old rule is either revised or retired with reasoning logged — no silent drift.
4. **Acceptance tests.** Sixteen tests defined what correct protocol execution looked like (schema validation, read-before-write ordering, contradiction detection, retrieval relevance). All sixteen passed on first implementation.

## Alternatives Considered

**Manual lessons file (status quo).** Rejected — this was the failing system. Operator-as-writer bottleneck guaranteed staleness.

**Free-form prose journals.** Rejected — no schema means no programmatic match against current build context. Retrieval relevance degrades the moment the corpus grows past what fits in working memory.

**Vector store / semantic retrieval.** Rejected for current corpus size (~36 files). Adds infrastructure cost without obvious gain at small scale. Acceptance test 14 (retrieval relevance) is the canary; if it fails on a future build, semantic retrieval becomes the next option. Threshold to reconsider: ~200 files, or first acceptance failure on relevance.

**Continual learning (gradient updates to the model).** Out of scope — no fine-tuning capability in the deployed stack. Would also be the wrong tool: build-specific lessons need to be inspectable, versioned, and rollback-able. Weights aren't.

## Consequences

**Positive.**
- Cross-session knowledge moved from 100% dark to 100% addressable. The protocol read-step has no failure mode where a relevant prior lesson is silently skipped.
- First-pass build acceptance moved from 31% (4 of 13 builds) to 86% (6 of 7 builds) post-protocol. (See "Limit" below — three other changes shipped the same week.)
- The operator is out of the knowledge-transfer loop. Lessons compound without operator effort.
- Three documented solution files in `examples/solution-files/` show the protocol catching real regressions, including one case where the build agent generalized from a prior lesson and fixed a novel failure autonomously without operator input.

**Negative / honest limits.**
- *Causal isolation is impossible.* The protocol shipped the same week as the architecture audit, the universal-scorer refactor, and the acceptance-test standards. The 31% → 86% gain is over-determined; this ADR cannot claim KCP alone produced the rate change.
- *The solution files are narrow.* They encode build-specific knowledge, not broad reasoning patterns. This is not a general-purpose memory system. A frontier-lab equivalent would require richer retrieval, consistency management, and broader scope.
- *Retrieval is mechanical, not semantic.* Will degrade if the corpus grows past where mechanical file scans stay cheap. Reconsider at the threshold above.
- *Generalization beyond build execution is the open experiment.* The protocol currently runs on one agent (the build agent). Extending to the Outreach Drafter is the next test of whether Layer 4 is mechanism or Claude-Code-specific accident. Result due within six weeks of this ADR.
- *Goodhart risk on gate metrics.* The operator's manual application of complexity-weighted build counts and 30-day post-deploy defect traceback is the enforcement layer for the safeguards. If the operator stops applying them, the gate metrics drift toward looking good rather than being good. Scaling past one operator requires moving this enforcement into the protocol itself, which this ADR does not specify.

## References

- [Case study, Layer 4](../case-study-full.md)
- `examples/solution-files/hardcoded-filenames.md`
- `examples/solution-files/forced-sale-file-finder.md`
- `examples/solution-files/pipeline-gate-false-positive.md`
- `examples/cc-spec-standards.md` (acceptance tests required, written before build code — concurrent change cited in "Limits")

---

*ADR-001 | Accepted April 5, 2026 | Authored as part of the public case study, April 2026*
