# ADR002 - Retain Captured Source Snapshots

When the C001 - Inbox processes a captured item, retain the raw material rather than deleting it: an integrated item is frozen as an immutable captured-source snapshot and a discarded item is kept as a tombstone. Deletion happens only on explicit, steward-directed purge.

## Context

Provenance is the reason the Inbox records an item's origin at capture (O001-R001). But O001-RSK002 covers two failure modes, not one: an external source can **change** after capture, or it can **disappear** entirely. Source revalidation (O001-R002) addresses the first — it detects drift — but detection needs something to diff against, and it does nothing for the second: once an external source is gone, a statement that cited only a mutable external reference is no longer verifiable by anyone. The architecture must decide what becomes of the raw inbox item once it has been processed, and whether captured material is durable evidence or transient scratch space.

## Options

- **Delete on integration**: once a statement is authored, drop the raw item. Simplest and smallest on disk, but destroys the only captured evidence — revalidation has no baseline, and a vanished external source leaves the statement unverifiable. Fails the second half of O001-RSK002.
- **Hash-only baseline**: keep a fingerprint of the captured material, not the material itself. Detects that a source changed, but cannot show *what* it was, and still leaves nothing to fall back on when the source disappears.
- **Retain the full captured snapshot** *(chosen)*: freeze the raw material as an immutable snapshot on integration and keep a tombstone on discard. Costs storage growth in exchange for durable, inspectable provenance.

## Decision

Adopt **full captured-snapshot retention**. On keep or merge, the Inbox freezes the raw material as an immutable captured-source snapshot that leaves the active backlog but persists; the integrated statement's source link resolves through it, and C006 - Source Revalidation diffs the live external source against it. On discard, the item is kept as a tombstone recording the rejection judgment. The corpus already commits to deprecation over deletion (O006); this applies the same principle at the point of intake, so the Inbox is not the one place Wiki throws knowledge away. The snapshot store is plain-text and version-controlled like the rest of the corpus (O005), so retention is cheap, diffable, and portable. Hard deletion exists only as an explicit, steward-directed purge for material that must not be retained.

## Consequences

- Enabling: closes the "source disappears" half of O001-RSK002 — provenance stays verifiable against the captured snapshot even when the external source is gone.
- Enabling: gives O001-R002 revalidation a concrete baseline to compare against, and lets a curator see what actually changed, not merely that something did. *(Refined by [ADR012 - Coarse Source Revalidation](ADR012-coarse-source-revalidation.md): "what changed" is now surfaced by presenting the captured snapshot alongside the live source for the steward, rather than by a machine line-diff.)*
- Enabling: reuses O005 portability — the snapshot store needs no bespoke persistence and travels with the bundle.
- Cost: the captured-source store grows monotonically; if volume ever becomes a real burden it is a retirement-review concern (O006-R002) applied to the archive, not a reason to delete on integration.
- Cost: sensitive material must be handled by an explicit purge path, adding a deliberate exception to the otherwise absolute retention guarantee.
