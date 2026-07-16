# C007 - Currency Tracking

Records when each concept last meaningfully changed and maintains a dated change history for each part of the corpus, materialized as in-corpus artifacts so recency and history travel with the bundle.

C007 is a **write-triggered recorder**, not a judge: it records the event a caller declares — it never diffs a concept's before/after content to decide for itself whether a change was meaningful. This is sound only because every corpus-mutating path is itself a deliberate, steward-directed operation (C003's `author`, C010's `declare_charter`/`revise_charter`, and C008's `deprecate`/`retire`/`relocate`) — the architecture's "intent in, work out" principle already rules out the kind of silent or cosmetic hand-edit that would need a meaningfulness filter. C007 fulfills recorded recency (O002-R001) and change history (O002-R002) outright. Unlike C005's graph or C006's and C010's findings — all recomputed on demand and never persisted — C007's two artifacts are durable corpus writes, so a consumer with no Wiki tooling can read a concept's recency and history straight off the files ([ADR008](../drs/ADR008-recency-materialization-and-log-scoping.md)).

## Data model

- **Recency (`timestamp`)** — the ISO-8601 `timestamp` frontmatter field (OKF §4.1) on any concept C007 has recorded a change for. C007 owns writing and maintaining its value, set to the moment of the recording call rather than any content-derived time; C004 owns the field's shape (presence and ISO-8601 form).
- **Change-log entry** — an OKF `log.md` entry (§7): date-grouped, newest first, `**<Kind>**: <prose>` naming the concept with a bundle-relative link, rendered and parsed through C004's log-entry helper. Wiki fixes a **closed kind vocabulary** — `Creation`, `Update`, `Deprecation`, `Retirement` — a Wiki convention layered on OKF §7's leading-bold-word convention (itself non-normative); a merge is recorded as `Update` with the merge described in prose, not a fifth kind. `Retirement` is the sole exception to "naming the concept with a bundle-relative link": the concept's file is gone by the time it's recorded, so the entry names it by title in plain text.
- **Directory log (`log.md`)** — one per directory containing at least one concept whose change C007 has recorded, holding entries only for concepts directly in that directory. Never a cascading or bundle-wide aggregate ([ADR008](../drs/ADR008-recency-materialization-and-log-scoping.md)); a directory with no recorded change has no `log.md` at all, mirroring C005's "no index.md with nothing to list" precedent.
- C007 owns no artifact beyond these two OKF-native ones — no separate database, cache, or index of "current recency" or "full history"; both are exactly what's on disk.

## Interfaces

C007 exposes an in-process face. `record` is its only corpus mutation; every other call is read-only.

- **`record(concept_id, kind, summary) → { timestamp, log_entry }`** — the single write entry point, mirroring C005's `reindex` as the sole mutation a writer triggers. Sets the concept's `timestamp` (via C004) to the moment of the call and appends one dated entry — `kind ∈ { Creation, Update, Deprecation, Retirement }`, `summary` the prose — to the `log.md` in the concept's own directory (via C004), except `Retirement`, which has no concept left to own a directory (see Edge cases). Called as the terminal step of any operation that mutates a concept's content or status: C003 after `author` (`Creation` for `keep-new`, `Update` for `merge`), C010 after `declare_charter` (`Creation`) or `revise_charter` (`Update`), and C008 after `deprecate` (`Deprecation` fresh, `Update` on edit), `retire` (`Retirement` for the retired concept itself, plus `Update` against every referrer whose link it repairs), and `relocate` (`Update` against both the moved concept and every referrer whose link it retargets).
- **`recency(concept_id) → timestamp | none`** — reads the concept's current `timestamp` via C004. `none` when the concept has never been through `record` (authored before C007 existed, or a writer that bypassed the contract).
- **`history(concept_id) → log_entry[]`** — every entry recorded for one concept, read from its directory's `log.md` (via C004) and filtered to entries naming that concept, newest first. Scoped to the concept's *current* directory only — see Edge cases for the relocation case this misses.
- **`history_all(scope?) → log_entry[]`** — the aggregated, newest-first change history for `scope` (a directory) or, when omitted, the whole bundle. Computed on demand by scanning every `log.md` under `scope`; nothing is persisted for this view ([ADR008](../drs/ADR008-recency-materialization-and-log-scoping.md)).

## Behavior

- **Materialize both, compute neither — except the aggregate view.** `timestamp` and log entries are durable corpus writes, not derived on read. `history_all` is C007's one on-demand computation, an aggregation over already-materialized data, not a judgment call the way C002's overlap scan or C010's reconciliation are.
- **Record what the caller declares.** `record` takes `kind` and `summary` from the calling writer; C007 performs no content diff to decide whether the change was "meaningful" — see the responsibility statement above for why that's sufficient.
- **Write-triggered, not polled.** The same contract ADR006 established for C005's `reindex`: C007 does not watch the corpus, so a writer that mutates a concept and skips `record` leaves that concept's recency stale until the next recorded event — a stewardship gap to catch via Lint/Survey, not a runtime guarantee C007 enforces.
- **One event, one entry, one file plus one field.** A single `record` call writes exactly the calling concept's `timestamp` and appends exactly one entry to exactly one `log.md` (its own directory's) — never more, regardless of the concept's directory depth ([ADR008](../drs/ADR008-recency-materialization-and-log-scoping.md) rules out cascade).
- **Append-only log, replace-in-place timestamp.** `log.md` only ever gains entries; `record` never edits or removes a prior one, including when re-recording the same concept. `timestamp` is replaced with each call — only the latest value survives there, and `history` is where the full trail lives.

## Edge cases

- **Concept never recorded** (authored before C007 existed, or a writer bypassed the contract): `recency` returns `none`; `history` returns `[]` — not an error. The first `record` call establishes both.
- **Directory with no recorded changes**: no `log.md` exists; `history_all` scoped there returns `[]`.
- **Merge (C003)**: recorded once, against the surviving target concept's directory, `kind = Update`; the absorbed material leaves no log trail of its own since it was never an authored concept in its own right.
- **Concept retired (C008's `retire`)**: `record` is called with `kind = Retirement` against the retired concept's *last* directory (its `log.md` still receives the entry even though the concept file is gone), naming it by title with no working link, since there is nothing left for a link to resolve to. Every referrer C008 repairs gets its own `Update` entry, recorded normally against its own (unaffected) directory.
- **Concept relocated to a different directory (C008's `relocate`)**: prior entries stay in the old directory's `log.md`, since entries are never rewritten or moved; the relocation itself is recorded as a fresh `Update` entry in the *new* directory's `log.md`. `history(concept_id)`, scoped to the concept's current directory, will not surface pre-relocation entries — a known imprecision, acceptable at the low-volume, single-steward scale — `history_all` at the bundle root still surfaces every entry regardless of which directory holds it. Every referrer whose link `relocate` retargets gets its own `Update` entry.
- **Two events on the same concept on the same calendar day**: both entries land under the same date heading in the same `log.md` — OKF §7's own example already shows multiple entries per date; no merging or dedup.
- **Charter revised in place (C010)**: `record` is called against the one charter concept's existing id and directory — no new file, consistent with C010's revise-in-place contract.

## Relationships

- **C003 - Integration Authoring**: calls `record` as the terminal step of `author`, resolving the question C003's own document left open — `Creation` when `direction.mode = keep-new` mints a new concept, `Update` when `mode = merge` folds into an existing one. C003 does not set the initial `timestamp` itself; C007 owns that value entirely.
- **C008 - Lifecycle & Retirement**: calls `record` as the terminal step of `deprecate` (`Deprecation` fresh, `Update` on edit), `retire` (`Retirement` for the retired concept, `Update` for every referrer it repairs), and `relocate` (`Update` for the moved concept and every referrer it retargets). `retire` and `relocate` are the only operations in the system where a single steward-directed call triggers more than one `record` call — each still individually one-event-one-entry, per Behavior. Also reads `recency`/`history` as its staleness signal in `retirement_candidates`, alongside C006's drift findings (O003-R002).
- **C010 - Charter**: calls `record` after `declare_charter` (`Creation`) and `revise_charter` (`Update`), so the charter concept's recency and history flow through C007 exactly like any other concept, matching C010's own claim that it "gets recency and change history from C007."
- **C004 - OKF Conformance**: C007's sole path to corpus content — the frontmatter helper to write and read `timestamp`, the log-entry helper to render and parse `log.md`. C007 encodes no OKF structural literal; C004 checks `timestamp`'s shape/presence and `log.md`'s structure, while writing and maintaining their values is C007's.
- **C005 - Index & Navigation**: no relationship. `index.md` and `log.md` are separate reserved filenames (OKF §3.1) with separate owners; neither reads or writes the other's artifact.
- **Boundary**: C007 owns recency and change history only — writing and reading `timestamp` and `log.md` entries for a change a caller declares. It does not decide whether a change is meaningful enough to record (the calling writer's own contract already guarantees that), judge staleness or retirement (C008), or define the OKF format (C004).

## Success criteria

- **Recency recorded**: after any `record` call, `recency(concept_id)` returns that call's timestamp — verifiable by recording and immediately reading it back.
- **History completeness (current directory)**: `history(concept_id)` returns exactly the entries recorded for that concept while it has lived in its current directory, newest first — verifiable by recording two events and confirming both appear in order.
- **One write per event**: a single `record` call mutates exactly one concept's `timestamp` and appends to exactly one `log.md` — verifiable by diffing the corpus before and after and confirming no other file changed.
- **No mutation on read**: `recency`, `history`, and `history_all` never write to the corpus.
- **Append-only log**: re-recording a concept never removes or edits a prior entry in its directory's `log.md` — verifiable by recording twice and confirming both entries persist.
- **Idempotent absence**: a concept or directory with nothing ever recorded returns `none` / `[]` rather than an error.

## Notes

- The closed change-kind vocabulary (`Creation` / `Update` / `Deprecation` / `Retirement`) is a Wiki convention layered on OKF §7's non-normative leading-bold-word convention — recorded here rather than in a separate ADR, the same treatment C004 gave the kebab-case slug convention; promote it if it becomes contested.
- Materializing both artifacts, and scoping `log.md` per directory with no cascade — computing any wider view on demand instead — are captured with alternatives and consequences in [ADR008](../drs/ADR008-recency-materialization-and-log-scoping.md).
- `Retirement`'s link-less log entry and `retire`/`relocate`'s multi-`record` fan-out follow from [ADR009](../drs/ADR009-retirement-relocation-and-link-repair-model.md), which reinstated real deletion and relocation as C008 operations.
- Currency tracking has no steward-facing operation of its own: `history_all` backs a Survey-style view of what changed recently, and `recency` is shown alongside any concept in Query.

## See Also

### Architectural Decision Records

- [ADR008 - Recency materialization and per-directory log scoping](../drs/ADR008-recency-materialization-and-log-scoping.md)
- [ADR009 - Retirement, relocation, and link-repair model](../drs/ADR009-retirement-relocation-and-link-repair-model.md)
