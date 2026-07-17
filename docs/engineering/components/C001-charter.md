# C001 — Charter

> **Status:** interface frozen (Chunk A). **Full behavioral spec: Phase 3 — pending.**

**Role.** The required, in-corpus scope referent — the corpus's declared purpose and
scope, read by every scope/significance/dedup/coverage judgment.

**Owns.** The charter policy anchor (`{ purpose, [ScopeArea] }`), in-corpus and
revisable-in-place; may be provisional.

**Frozen interface.** See [INTERFACES.md](../INTERFACES.md) § C001. Do not widen the
surface without a cross-component change.

**Requirements owned.** O008-R001 (see [REQUIREMENT-MAP.md](../REQUIREMENT-MAP.md)).

**Key relationships.** Depends on nothing (foundation). Read by C002, C007, C008, C011,
C012. Written by the steward (declare/revise).

**Decisions.** Charter is **required** (PDR004), reversing the old optional stance — the
charter-less degradation path is consciously dropped (see
[DECOMPOSITION.md](../DECOMPOSITION.md#consciously-dropped)).

**Phase-3 spec must add:** the OKF representation of the charter, the declare/revise
flow, and success criteria tied to O007/O008 being well-defined (scope always has a
referent).
