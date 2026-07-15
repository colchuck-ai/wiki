# C008 - Lifecycle & Retirement

Marks a concept as deprecated or superseded — with a reason and, where one exists, a pointer to its replacement — rather than hard-deleting it, and supports periodically identifying retirement candidates, so obsolete knowledge is retired without dead-ending the graph.

Lifecycle & Retirement is a standing capability over the corpus. It owns the deprecation lifecycle and the periodic retirement review; it never hard-deletes. It is invoked by C003 - Integration Authoring during a supersede reconciliation, by C001 - Inbox on a provenance-affecting purge, and runs as the retirement-review check of the **Lint** operation (see [ADR010 - Curation Agent Skill and Operations](../drs/ADR010-curation-agent-skill.md)).

## Data model

Lifecycle state is held as in-corpus, plain-text markers on the concept, authored following C004 - OKF Conformance conventions (OKF §4.1 producer-defined keys), so they travel with the bundle like the rest of the corpus (O005):

- **Deprecation marker** — records that a concept is deprecated or superseded, carrying a reason and, where one exists, a replacement pointer (a bundle-relative `/`-rooted link, OKF §5.1).
- **Provenance-purged marker** — records that a concept's captured source was purged (see [ADR013 - Provenance-Purge Cascade](../drs/ADR013-provenance-purge-cascade.md)), carrying a reason; signals the concept is no longer source-backed.

These are markers on the concept, not a separate store or external bookkeeping.

## Behavior

**Deprecate / supersede.** Mark the concept in place with a reason and, where one exists, a replacement pointer. Inbound links are resolved first (via C005 - Index & Navigation) so no inbound reference dead-ends. Invoked by C003 - Integration Authoring on a supersede merge reconciliation.

**Retirement review** (periodic, in Lint). Because Wiki captures no usage or reliance signal (see [PDR003 - Reliance Is Steward-Supplied](../../product/drs/PDR003-reliance-steward-supplied.md)), "no longer relied upon" is not system-measured. Retirement candidates are assembled from **measurable** signals — staleness (C007 - Currency Tracking `timestamp` age), source drift (C006 - Source Revalidation), superseded-but-not-yet-cleaned concepts, and content the scaffold scope no longer frames (C010 - Scaffold Survey drift) — and presented for the steward's retirement judgment. The tool supplies the signals; the steward supplies the reliance judgment; retirement is executed as deprecation, never a hard delete.

**Provenance-purge cascade.** When C001 - Inbox purges the captured source of an already-integrated item, mark the concept(s) it fed (via the item's `concept` backref) provenance-purged with a reason (per ADR013).

## Edge cases

- **No replacement exists for a superseded concept**: deprecate with a reason and no pointer — O006-R001 requires a pointer only "where one exists."
- **Retirement candidate still on-scope and current**: presented, not auto-retired; the steward may keep it.
- **Concept with inbound links proposed for retirement**: inbound links are resolved via C005 first, so deprecation never leaves a dangling reference.

## Relationships

- **C003 - Integration Authoring**: invokes C008 to deprecate a superseded statement, with a reason and a replacement pointer, during a supersede reconciliation.
- **C005 - Index & Navigation**: consulted for inbound links before a concept is deprecated or relocated, so traversal never dead-ends.
- **C006 - Source Revalidation**: source drift is a retirement-candidate signal; a provenance-purged concept has no revalidation baseline.
- **C007 - Currency Tracking**: `timestamp` staleness is a retirement-candidate signal, and each deprecation is a change event C007 records.
- **C001 - Inbox**: a provenance-affecting purge cascades a provenance-purged mark to the dependent concept(s).
- **C010 - Scaffold**: Survey scope-drift feeds the retirement-candidate signals.

## Success criteria

- A replaced concept is marked deprecated with a reason and, where one exists, a replacement pointer — never hard-deleted (O006-R001).
- Retirement review surfaces candidates from measurable signals (staleness, source drift, supersession, scope-drift) for the steward's judgment, rather than leaving obsolete concepts to accumulate unnoticed (O006-R002).
- No deprecation or retirement leaves an inbound link dangling (O003-R004).
- A concept whose captured source was purged is visibly marked, so no consumer relies on it as source-backed.

## See Also

### Architectural Decision Records

- [ADR004 - Merge Reconciliation Steering](../drs/ADR004-merge-reconciliation-steering.md)
- [ADR013 - Provenance-Purge Cascade](../drs/ADR013-provenance-purge-cascade.md)

### Change Records

- [CR009 - Lifecycle & Retirement Component](../../crs/CR009-lifecycle-retirement-component.md)
- [CR010 - Reliance Signal Reframe](../../crs/CR010-reliance-signal-reframe.md)
- [CR013 - Provenance-Purge Cascade](../../crs/CR013-provenance-purge-cascade.md)
