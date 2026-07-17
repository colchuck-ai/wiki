# C004 — Source Retention

> **Status:** interface frozen (Chunk A). **Full behavioral spec: Phase 3 — pending.**

**Role.** Hold the durable/live source pair for every citation and compute drift.

**Owns.** Durable source snapshots (version-controlled) + distinct live-origin locators;
drift computation. One substrate serving both provenance (O001) and fidelity (O009).

**Frozen interface.** See [INTERFACES.md](../INTERFACES.md) § C004 (`retain`, `resolve`,
`check_drift`). `durable_ref` is the resolution target and is never substituted by
`live_locator`.

**Requirements owned.** O001-R002, O001-R003. See
[REQUIREMENT-MAP.md](../REQUIREMENT-MAP.md).

**Key relationships.** `retain` called by C008 at **authoring time** (not intake — C006
never calls C004, keeping C004 entirely corpus-side). `resolve` read by C008 (grounding)
and C011. `check_drift` used by the C005 currency refresh.

**Decisions.** [ADR003](../drs/ADR003-citation-two-reference-model.md) (two-reference model
+ authoring-time capture; supersedes old CR001/ADR007).

**Phase-3 spec must add:** the snapshot storage form, live-fetch + drift-diff behavior,
idempotency, and success criteria tied to the O001 proxy (citations resolving to a
durable form).
