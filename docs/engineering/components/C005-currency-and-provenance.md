# C005 — Currency & Provenance

> **Status:** interface frozen (Chunk A). **Full behavioral spec: Phase 3 — pending.**

**Role.** The append-only history of the corpus and its currency picture.

**Owns.** The decision-provenance log (plain-text, in-corpus, scoped so history travels
with the material — not central), drift-status materialization, per-concept currency.

**Frozen interface.** See [INTERFACES.md](../INTERFACES.md) § C005 (`record`, `history`,
`recency`, `currency`). Single writer of the provenance store — callers **report**, never
write it directly.

**Requirements owned.** O002-R001, O002-R002. See
[REQUIREMENT-MAP.md](../REQUIREMENT-MAP.md).

**Key relationships.** All mutators report dispositions via `record`. Materializes drift
status from `C004.check_drift`. Read by C011 (retirement signals), consumers (currency).

**Decisions.** [ADR004](../drs/ADR004-machine-readable-signal-placement.md) (the log is
portable/scoped; inline signals stay in the concept file, written by C008).

**Phase-3 spec must add:** the log's physical scoping, how recency is derived vs the
inline stamp, the drift-refresh cadence, and success criteria tied to the O002 proxy
(recency + drift up to date relative to sources).
