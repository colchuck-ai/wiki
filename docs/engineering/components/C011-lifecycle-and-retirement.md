# C011 — Lifecycle & Retirement

> **Status:** interface frozen (Chunk A). **Full behavioral spec: Phase 3 — pending.**

**Role.** The retirement/relocation decision-maker: surface stale/superseded concepts
and drive their marking and link repair.

**Owns.** Nothing persistent — a decision module. Owns the referential-integrity-
sensitive moves (retire **and** relocate).

**Frozen interface.** See [INTERFACES.md](../INTERFACES.md) § C011 (`review() ->
[RetirementCandidate]`, then drives C008). Mutates the corpus **only** by driving C008
intent verbs — never edits files directly.

**Requirements owned.** O003-R002 (retirement review), O004-R003 (surface half), O005-R003
(link repair). See [REQUIREMENT-MAP.md](../REQUIREMENT-MAP.md).

**Key relationships.** Reads C005 (currency/drift signals) + in-corpus supersession
markers. Uses C009.inbound_links before retire/relocate. Drives C008
(deprecate/relocate/repair_links) within the envelope (C002.evaluate); escalates
ambiguous cases via C002.

**Decisions.** Repair-to-successor + deprecate-vs-delete settled in-component (settles old
ADR009/ADR012); no ADR.

**Phase-3 spec must add:** the candidate-detection signal set, the deprecate/supersede/
relocate decision rules, and success criteria tied to the O003 proxy (flagged-and-
unresolved rate falling).
