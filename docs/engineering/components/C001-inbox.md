# C001 - Inbox

The single, producer-agnostic capture point where raw material is recorded — with its origin — the moment it arrives, retained as durable provenance after processing, and where the count and age of un-integrated items are exposed.

The Inbox is the only entry point into Wiki. It records what arrived, from whom, and when, and it never destroys captured material as a side effect of processing: an item leaves the active backlog by changing state, not by being deleted.

The Inbox is a Wiki-owned store that sits outside the OKF conformance boundary (raw material is not conformant knowledge yet), so its layout is Wiki's to define — it is not governed by the OKF conventions C004 - OKF Conformance owns.

## Data model

Each inbox item is one plain-text file under a `raw/` directory, named by a date-prefixed local id so items sort by capture age. State is a frontmatter field; **nothing ever moves** and the file persists for the life of the reference (the modern email-inbox model — one accumulating store, state as a flag, backlog as a filter).

```
raw/2026-07-13-okf-spec.md
---
id:       2026-07-13-okf-spec
origin:   okf-reference-agent          # the producer (human or agent)
source:   https://okf.example/spec     # external reference, or: none
captured: 2026-07-13
status:   pending                       # pending | integrated | discarded
---
<raw captured material — the body>
```

- **Inbox item**: the raw captured material (body) plus capture metadata (`origin`, `source`, `captured`).
- **State** (`status`): `pending`, `integrated`, or `discarded`. Only `pending` items count toward the backlog; a transition is a frontmatter edit, never a file move.
- **Captured-source snapshot**: once `status: integrated`, the file's body is frozen and the file itself *is* the immutable snapshot — the artifact an integrated statement's source link resolves to (`raw/<id>.md`) and the baseline C006 - Source Revalidation diffs the live external source against. The file does double duty: intake record and durable provenance.
- **Non-text captured source**: when the originating source is inherently non-text (an image, a PDF, a data file), its captured form is retained as a version-controlled binary **asset** (for example under `raw/assets/`) colocated with the item and referenced from it; that asset is the immutable captured-source snapshot in place of a text body. Curated concepts stay plain-text; only retained source provenance may be binary (see [PDR002 - Provenance Assets Exception](../../product/drs/PDR002-provenance-assets-exception.md) and [ADR008 - Captured Source Assets](../drs/ADR008-captured-source-assets.md)).

Invariant: the `raw/` directory is append-and-annotate only, so a file's git history is a convenient record of the item's inbox lifecycle. It is not the O004 change-history system of record, though: recency and change history are materialized in-corpus as `timestamp`/`log.md` by C007 - Currency Tracking, because git history does not survive OKF's tarball/subdirectory distribution (see [ADR006 - In-Corpus Currency](../drs/ADR006-in-corpus-currency.md)).

## Behavior

An item settles into exactly one terminal `status` and never re-enters the backlog. Every transition is a frontmatter edit in place; the file does not move:

- **Captured → `pending`.** Any producer — a human, an automated producer, or an agent consuming the reference that files a synthesized answer back — deposits raw material; the Inbox writes a new file under `raw/` with its `origin`, `source`, and `captured` date, `status: pending`. It now counts toward the backlog.
- **`pending` → `integrated`** (steward directs keep or merge). `status` flips to `integrated`; `integrated: <date>` and `concept: <okf-id>` (a backref to the concept it fed) are added. The body is now frozen and serves as the captured-source snapshot. Retained, not deleted, so provenance survives even if the external source later changes or disappears.
- **`pending` → `discarded`** (steward directs discard). `status` flips to `discarded`; `discarded: <date>` and `reason:` are added. The body is retained as a tombstone. Discard records a judgment; it does not erase what was considered.

No transition hard-deletes captured material. The only removal path is an explicit, steward-directed purge (see Edge cases), never an automatic consequence of processing.

## Edge cases

- **Explicit purge of sensitive material**: a producer deposits material that must not be retained (e.g. secrets, personal data). The steward may direct a deliberate purge, which replaces the file body with a tombstone note and adds `purged: <date>` and `purge-reason:` while leaving the frontmatter record intact. For a non-text captured asset, purge also removes the asset bytes, leaving only the tombstone record. Purge is a flag orthogonal to `status`, not a fourth state. This is the sole exception to retention and is never the default path.
- **Non-text source at capture**: the source is an image, PDF, or data file. It is captured as a version-controlled asset (see Data model) rather than forced into a text body, so its captured form is retained at full fidelity for provenance and revalidation.
- **Re-deposit of already-integrated material**: the same source arriving again is captured as a new pending item; adjudication (C002) decides merge-versus-discard against what the corpus already holds.
- **Origin unknown at capture**: the item is still captured with whatever origin metadata exists; a missing external reference is recorded as such rather than blocking intake, so nothing is lost before triage.

## Relationships

- **C002 - Scope Triage**: hands each pending item to triage for adjudication against the declared scope.
- **C003 - Integration Authoring**: supplies the captured-source snapshot so an integrated statement can carry a source link that resolves to the material as captured, not only to a mutable external reference.
- **C006 - Source Revalidation**: exposes the captured-source snapshot as the baseline against which the live external source is revalidated.

## Success criteria

- No captured item is ever lost or overwritten as a side effect of triage or integration; every terminal transition preserves the content (integrated snapshot or discard tombstone) unless an explicit purge was directed.
- The backlog count and age reflect only `pending` items — an integrated or discarded item drops out of the metric immediately on transition.
- For every integrated statement, the captured-source snapshot it derives from is retrievable, so provenance is verifiable even after the external source changes or disappears.

## See Also

### Architectural Decision Records

- [ADR002 - Retain captured source snapshots](../drs/ADR002-retain-captured-source-snapshots.md)
- [ADR003 - Inbox storage layout](../drs/ADR003-inbox-storage-layout.md)
- [ADR008 - Captured Source Assets](../drs/ADR008-captured-source-assets.md)

### Change Records

- [CR001 - Captured-source retention](../../crs/CR001-captured-source-retention.md)
- [CR004 - Captured Source Assets](../../crs/CR004-captured-source-assets.md)
- [CR006 - Query Feedback Loop](../../crs/CR006-query-feedback-loop.md)
