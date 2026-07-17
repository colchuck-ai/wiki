# C007 — Triage

> **Status:** interface frozen (Chunk A). **Full behavioral spec: Phase 3 — pending.**

**Role.** The disposition decider: scope + duplication + significance → a routing
decision.

**Owns.** Nothing persistent — a pure decision module (charter + item + corpus →
`TriageOutcome`), which is also its whole test surface.

**Frozen interface.** See [INTERFACES.md](../INTERFACES.md) § C007
(`triage(RawMaterial) -> TriageOutcome`, where `TriageOutcome = integrate | hold_aside |
merge(into) | escalate`).

**Requirements owned.** O003-R003 (duplication), O003-R004 (significance), O008-R002
(scope-aware triage). See [REQUIREMENT-MAP.md](../REQUIREMENT-MAP.md).

**Key relationships.** Reads C001 (charter) and C009 (overlap lookup). Consequential
calls go through C002.evaluate; ambiguous ones escalate. Routes via C006.hold_aside /
C008.integrate|merge — executes nothing itself.

**Decisions.** Disposition set settled in-component (settles old ADR004); no ADR.

**Phase-3 spec must add:** the scope/dup/significance judgment model (all relative to
the charter), the overlap-detection approach, and success criteria tied to O003 (dedup,
significance) and O008 (scope) proxies.
