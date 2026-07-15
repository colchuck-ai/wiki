# C001 - Ingestion Queue

The single producer-agnostic intake point: records each ingestion task with a reference to the material to distill, ensures that source is durably held — referenced in place when it is version-controlled alongside the corpus, or captured as a snapshot when it is not — and exposes the backlog of un-triaged items.

C001 is the head of the curation pipeline and the realization of the product's **Ingest** intake step. It owns one job: get raw material *recorded and durably held* the moment it arrives, so nothing is lost before triage (O007-R001) and no cited source can rot before it is even adjudicated (O001-R002). It makes no admission decision — scope, overlap, and significance are judged downstream by C002 - Triage. The queue is a **staging area outside the corpus bundle**, so un-triaged material never appears in the body of knowledge, its index, or its graph until C003 - Integration Authoring writes it in. The queue's own record format is C001's, not OKF; C004 governs the corpus, not the staging area.

## Data model

C001 owns two staging artifacts, both plain-text and git-tracked from the moment of intake (satisfying O006-R001 and the version-control-native constraint for the staging area):

- **Ingestion task.** One file per submitted item under the staging tree (e.g. `queue/T<NNN>-<slug>/task.md`), carrying:
  - `id` — a monotonic queue-local task id (`T<NNN>`), never reused after a task is discarded so historical references stay unambiguous. This is a runtime staging id, distinct from any intent logical ID.
  - `status` — `untriaged` on creation; advanced downstream (`admitted` / `rejected` / `held`) by C002 per this schema. The backlog is exactly the set of `untriaged` tasks.
  - `origin` — the original locator the material came from (URL, external path, or a note for a non-locatable origin). Recorded for provenance and as C006's later live-source handle; never used to gate admission.
  - `source` — the durable-holding record (below): either an in-place reference or a captured snapshot.
  - `submitted-at` — ISO-8601 capture time.
  - `producer` — optional, free-form note of who/what submitted (person, tool, or agent). Recorded, never adjudicated on — intake is producer-agnostic.
  - Body — the submitter's free-text note: why it matters, a distillation hint, anything to carry to triage.
- **Durable-holding record** (`source`), one of two shapes:
  - **In-place reference** — when the material is already version-controlled alongside the corpus: a repo-relative `path`, plus a `content-hash` captured at intake. No bytes are copied.
  - **Captured snapshot** — when the material cannot be referenced durably in place (external URL, remote or out-of-repo file): a `snapshot` path to an immutable artifact under the task's staging dir, plus `media-type`, `content-hash`, and `captured-at`. Textual sources are stored as UTF-8 text; non-text sources are stored as a version-controlled binary asset — the bounded provenance fallback the architecture permits.

The `content-hash` is load-bearing: it is the baseline C006 - Source Revalidation later diffs the live source against to detect drift, and the fingerprint C002 - Triage can use for its duplication check. C001 records it; it does not interpret it.

## Interfaces

C001 exposes an in-process face for submission and backlog access. All corpus side effects are confined to the staging tree.

- **`submit(origin, [material], [note], [producer]) → task_id`** — the single producer-agnostic entry point. Creates the task, performs durable holding synchronously (see Behavior), and returns the task id. `material` is optional when `origin` is already a repo-relative path (in-place reference); otherwise C001 captures a snapshot from `origin` (or from supplied `material`).
- **`list_backlog() → task[]`** — the un-triaged backlog. Consumed by C002 - Triage and surfaced through the Ingest/Survey operations.
- **`get_task(id) → task`** — the full task record.
- **`durable_ref(task) → { path | snapshot, media-type, content-hash, origin }`** — resolves the durably-held source for downstream use: C002 inspection, C003 promotion at authoring, C006 drift baseline.
- **`discard(task)`** — removes the task and its staged artifacts. Invoked on rejection. Because staging is outside the corpus bundle, discard touches nothing in the body of knowledge.

**Promotion boundary.** When C002 admits an item, its snapshot is promoted from staging into the corpus provenance area and cited as part of authoring — a C003 action using C004's citation/link conventions. C001 exposes the staged artifact via `durable_ref`; whether the physical move is a C001 helper C003 calls or a C003 operation is an interface detail to settle when C003 is specced. Until promotion, the staging record is the sole durable holding.

## Behavior

- **Durable at intake.** Holding happens synchronously inside `submit`, before the task is exposed — so the source cannot rot between intake and triage. This is the concrete guard for O001-RSK002 (source rot) applied at the earliest possible point.
- **Never lose before intake.** A submission always yields a recorded task, even when durable capture fails (see Edge cases). Material is never silently dropped — the failure is flagged for the steward, not thrown away. This is the guard for O007-RSK001 (lost before intake).
- **Reference in place, snapshot only as fallback.** C001 prefers an in-place reference whenever the material is already version-controlled alongside the corpus, and captures a snapshot only when it is not — directly mirroring O001-R002's wording and avoiding duplicate bytes for material the repo already holds.
- **Immutable snapshots.** A captured snapshot's bytes never change after capture; its `content-hash` is stable. Re-capturing an origin creates a new artifact rather than mutating the old one, so C006's drift baseline is trustworthy.
- **No admission logic.** C001 applies no scope, overlap, or significance test. Every submission enters the backlog; judgment is C002's.

## Edge cases

- **Origin unreachable at intake** (e.g. a URL returns 404): the task is still created and enters the backlog, flagged as capture-failed with the `origin` recorded, so the steward can supply the material or discard it — the item is never lost, but it is not falsely marked durably held.
- **Origin neither referenceable in place nor snapshottable** (a live system, an in-person conversation): the task records the `origin` descriptively and marks durable holding N/A for the steward's judgment. Provenance is best-effort here, by nature of the source.
- **Non-text source** (PDF, image): captured as a version-controlled binary asset — the bounded fallback. C004's plain-text-concept `error` rule explicitly excludes declared snapshot assets, so this does not later read as a conformance failure.
- **Duplicate submission of the same material**: C001 does not deduplicate — it records the task and its `content-hash`. Detecting overlap and deciding merge/keep/discard is C002's duplication check (O003-R003).
- **In-place reference whose target later moves or is deleted**: C001's reference is path-based; the `content-hash` captured at intake lets downstream detect that the target changed or vanished. Resolving broken references is C005/C006 territory, not intake's.
- **Task rejected at triage**: `discard` removes the staged task and snapshot; because nothing was promoted into the corpus, no body-of-knowledge cleanup is needed.

## Relationships

- **C002 - Triage**: reads `list_backlog` and each task record, records the disposition into `status` per C001's schema, may use `content-hash` for its duplication check, and calls `discard` on rejection. The task file is the intake→triage interface artifact.
- **C003 - Integration Authoring**: on admission, consumes the task and its `durable_ref`; promotes the snapshot into the corpus provenance area and attaches the source citation (via C004). The reference relocation at promotion is part of authoring, not intake.
- **C006 - Source Revalidation**: uses the captured snapshot (or in-place reference) together with `origin` and `content-hash` as the baseline to detect drift against the live source — only after the item is authored into a concept.
- **C004 - OKF Conformance**: C004 governs the corpus, not the staging queue; C001's task format is its own. C004's conventions apply to the source citation only at authoring time (C003), not to the staging record.
- **Boundary**: C001 owns *intake and durable holding*. It does not adjudicate scope (C002/C010), detect duplication (C002), author concepts (C003), or repair references (C005/C006). It guarantees only that submitted material is recorded and durably held until judged.

## Success criteria

- **Nothing lost before intake**: every `submit` yields a durable, retrievable task record — including when capture fails, in which case the task is present and flagged rather than dropped.
- **Durable at intake (rot-proof)**: for an external source, after `submit` the material is retrievable from git independent of its origin — deleting or redirecting the origin leaves the snapshot resolvable. Testable by capturing, then breaking the origin, then resolving `durable_ref`.
- **Reference in place, no needless copy**: an already-version-controlled source yields a path reference plus a hash and copies no bytes; only non-in-place sources produce a snapshot.
- **Immutable snapshot**: a captured snapshot's `content-hash` never changes after capture; re-capture yields a new artifact.
- **Producer-agnostic single path**: one `submit` entry with no producer-type branch that gates admission — verifiable by submitting identical material as different producer types and getting identical handling.
- **Backlog fidelity**: `list_backlog` returns exactly the `untriaged` tasks; admitted and rejected tasks drop out.
- **Clean rejection**: `discard` removes a task's staged artifacts and leaves the corpus bundle byte-for-byte untouched (staging is outside the bundle).

## Notes

- The queue's staging location (`queue/` at repo root, sibling to the corpus bundle) and its task-record format are **Wiki choices** local to C001, deliberately outside OKF — recorded here rather than in a separate decision record. The intake staging model and the capture-then-promote durable-source flow are captured in [ADR002](../drs/ADR002-intake-staging-and-durable-source.md).
- The corpus-side provenance area that snapshots are promoted into at authoring, and its path convention, are settled with C003; C001 does not fix that layout.

## See Also

### Architectural Decision Records

- [ADR002 - Intake staging and durable-source capture](../drs/ADR002-intake-staging-and-durable-source.md)
- [ADR004 - Triage disposition model](../drs/ADR004-triage-disposition-model.md)
