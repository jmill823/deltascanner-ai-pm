# ADR-001: Knowledge Compounding Protocol (one-pager)

**Accepted April 5, 2026** | Layer 4 (Data)

## Context

Every build session started from zero. A build on city #12 could repeat a mistake city #3's build had already solved. The repo accumulated commits but not lessons. A manually maintained lessons file failed for the reason these always fail: the operator was the bottleneck on writing to it.

## Decision

Move the knowledge store out of the operator's hands and into the system's operating loop. After every build, the build agent writes a structured solution file to `docs/solutions/` with a canonical schema (failure class, symptom, root cause, prevention rule, affected files). Before every new build, the agent reads all prior solution files and summarizes the relevant ones in its plan — a required gate. When a new solution contradicts an existing one, the agent surfaces it as a BLOCKING flag rather than silently overwriting; the operator adjudicates. Sixteen acceptance tests defined correct execution. All sixteen passed first implementation.

## Alternatives (one line each)

Manual lessons file (status quo) — the failing system, operator-as-writer bottleneck. Free-form prose journals — no schema, no programmatic match against current build context. Vector store — premature at 36 files, reconsider at ~200 or first relevance failure. Continual learning (gradient updates) — fine-tuning out of scope and the wrong tool for build-specific, inspectable, rollback-able knowledge.

## Honest limit

Causal isolation is impossible. The protocol shipped the same week as three other structural changes (architecture audit, universal-scorer refactor, acceptance-test standards). The 31% → 86% first-pass build acceptance gain is over-determined; this ADR cannot claim KCP alone produced the rate change. The protocol currently runs on one agent. Whether Layer 4 generalizes — or remains build-agent-specific accident — is the open experiment.

## Revisit trigger

Reconsider mechanical retrieval at ~200 solution files or first acceptance-test-14 (relevance) failure.

---

**Full ADR** at [`architecture/adr-001-knowledge-compounding-protocol.md`](../architecture/adr-001-knowledge-compounding-protocol.md). Includes consequences in detail, three documented solution files, and references to acceptance-test standards.
