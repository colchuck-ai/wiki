# C012 — Coverage & Scope Review

> **Status:** interface frozen (Chunk A). **Full behavioral spec: Phase 3 — pending.**

**Role.** Read-only charter-vs-corpus surveys: find thin/absent scope areas (coverage)
and content that has drifted out of scope.

**Owns.** Nothing persistent — a survey/decision module.

**Frozen interface.** See [INTERFACES.md](../INTERFACES.md) § C012 (`coverage() ->
CoverageReport`, `scope() -> [ScopeDivergence]`). Read-only — produces reports and drives
C008/C002; never writes the corpus itself.

**Requirements owned.** O004-R003 (surface half), O007-R002 (coverage review), O008-R003
(scope reconciliation). See [REQUIREMENT-MAP.md](../REQUIREMENT-MAP.md).

**Key relationships.** Reads C001 (charter), C009 (index + dangling targets), C006
(held-aside count). Escalates via C002; scope flags executed via C008.flag. Mechanism
(metric vs agent-scan) is open, like C010.

**Decisions.** Coverage signal set (charter vs index + held-aside + dangling targets)
settled in-component; the old no-charter degradation branch is obsolete (charter now
required, PDR004). No ADR.

**Phase-3 spec must add:** the coverage + scope signal models and their mechanism, and
success criteria tied to the O007 (scope-area coverage; held-aside awaiting review) and
O008 (scope-divergent flagged) proxies.
