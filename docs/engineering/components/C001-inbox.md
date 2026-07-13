# C001 - Inbox

The single, producer-agnostic capture point where raw material is recorded — with its origin — the moment it arrives, retained as durable provenance after processing, and where the count and age of un-integrated items are exposed.

The Inbox is the only entry point into Wiki. It records what arrived, from whom, and when, and it never destroys captured material as a side effect of processing: an item leaves the active backlog by changing state, not by being deleted.

The Inbox is a Wiki-owned store that sits outside the OKF conformance boundary (raw material is not conformant knowledge yet), so its layout is Wiki's to define — it does not pass through C004 - OKF Conformance Adapter.

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

Invariant: the `raw/` directory is append-and-annotate only. A file's git history is therefore the item's full lifecycle, which also feeds change-history currency (O004).

## Behavior

An item settles into exactly one terminal `status` and never re-enters the backlog. Every transition is a frontmatter edit in place; the file does not move:

- **Captured → `pending`.** Any producer — human or automated — deposits raw material; the Inbox writes a new file under `raw/` with its `origin`, `source`, and `captured` date, `status: pending`. It now counts toward the backlog.
- **`pending` → `integrated`** (steward directs keep or merge). `status` flips to `integrated`; `integrated: <date>` and `concept: <okf-id>` (a backref to the concept it fed) are added. The body is now frozen and serves as the captured-source snapshot. Retained, not deleted, so provenance survives even if the external source later changes or disappears.
- **`pending` → `discarded`** (steward directs discard). `status` flips to `discarded`; `discarded: <date>` and `reason:` are added. The body is retained as a tombstone. Discard records a judgment; it does not erase what was considered.

No transition hard-deletes captured material. The only removal path is an explicit, steward-directed purge (see Edge cases), never an automatic consequence of processing.

## Edge cases

- **Explicit purge of sensitive material**: a producer deposits material that must not be retained (e.g. secrets, personal data). The steward may direct a deliberate purge, which replaces the file body with a tombstone note and adds `purged: <date>` and `purge-reason:` while leaving the frontmatter record intact. Purge is a flag orthogonal to `status`, not a fourth state. This is the sole exception to retention and is never the default path.
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

### Change Records

- [CR001 - Captured-source retention](../../crs/CR001-captured-source-retention.md)
