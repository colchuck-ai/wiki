# C009 — Discovery & Index

> **Status:** interface frozen (Chunk A). **Full behavioral spec: Phase 3 — pending.**

**Role.** The deterministic navigational structure over the corpus.

**Owns.** The navigable index + the reason-annotated cross-link graph, materialized from
the corpus (read-only over it). Machine-readable structure with a human rendering.

**Frozen interface.** See [INTERFACES.md](../INTERFACES.md) § C009 (`index`, `neighbors`,
`inbound_links`, `dangling`).

**Requirements owned.** O005-R001 (index), O005-R002 (graph — links written by C008),
O005-R003 (referential integrity). See [REQUIREMENT-MAP.md](../REQUIREMENT-MAP.md).

**Key relationships.** Reads the corpus. Provides `inbound_links` to C011 (before
retire/relocate) and `index`/`dangling` to C012 (coverage). Uses C003 for name/structure
validation.

**Decisions.** Index **materialize-vs-compute** is a component-internal choice hidden
behind this interface — deferred to this Phase-3 spec (settles old ADR006 at the right
altitude), not an architecture ADR.

**Phase-3 spec must add:** the materialize-vs-compute decision, the freshness/dry-run
query, rendering, and success criteria tied to the O005 proxy (index coverage,
cross-link density with reasons, absence of dangling links).
