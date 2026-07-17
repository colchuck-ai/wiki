# C010 — Question Retrieval

> **Status:** interface frozen (Chunk A). **Full behavioral spec: Phase 3 — pending.**

**Role.** Heuristic free-form lookup: a question in the consumer's own words → the
relevant concepts, for when they cannot guess the heading (vocabulary mismatch).

**Owns.** Nothing persistent, or a derived retrieval index — hidden behind the interface.

**Frozen interface.** See [INTERFACES.md](../INTERFACES.md) § C010
(`retrieve(question) -> [RankedConcept]`). Heuristic — may miss; ranked, not guaranteed.

**Requirements owned.** O005-R004. See [REQUIREMENT-MAP.md](../REQUIREMENT-MAP.md).

**Key relationships.** Read-only over the corpus. Distinct from C009 because it is
heuristic where C009 is deterministic.

**Decisions.** [ADR005](../drs/ADR005-question-retrieval-mechanism.md): the **mechanism is
open** behind the interface (agent-over-corpus vs a built search index) — chosen and
swappable in this Phase-3 spec without touching any other component.

**Phase-3 spec must add:** the chosen (initial) mechanism, its backing data if any, and
success criteria tied to the O005 true metric (time to locate), honestly heuristic.
