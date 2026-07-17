# C006 — Intake & Held-Aside

> **Status:** interface frozen (Chunk A). **Full behavioral spec: Phase 3 — pending.**

**Role.** Own all non-integrated material — the producer-agnostic raw queue and the
recoverable held-aside store.

**Owns.** Raw submissions (pre-triage) + the held-aside store (below-significance /
ambiguous-scope holds). Nothing is destroyed here; purge is a separate explicit act.

**Frozen interface.** See [INTERFACES.md](../INTERFACES.md) § C006 (`submit`, `next`,
`hold_aside`, `list_held_aside`, `restore`).

**Requirements owned.** O007-R001, O007-R003. See
[REQUIREMENT-MAP.md](../REQUIREMENT-MAP.md).

**Key relationships.** `submit` by any producer. `next`/`hold_aside` driven by C007.
`list_held_aside` read by C012 (part of the O007 proxy). `restore` is the reversal path
(deprecation over deletion).

**Decisions.** Held-aside is a separate recoverable store (not an in-corpus marker),
confirmed at the decomposition fork; no ADR (follows from deprecation-over-deletion +
held-aside material never having been a concept).

**Phase-3 spec must add:** the staging + held-aside data model, audit/reversal, the
purge boundary, and success criteria tied to the O007 proxy (nothing lost before intake;
held-aside count visible).
