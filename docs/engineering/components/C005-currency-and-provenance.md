# C005 — Currency & Provenance

**Role.** The append-only decision-provenance log of the corpus and the per-concept
currency picture derived from it — the single store everyone *reports* to and no one
writes directly.

## Capabilities

Tied to the frozen interface ([INTERFACES.md § C005](../INTERFACES.md#c005-currency--provenance-the-append-only-history) — signatures are authoritative there; not restated):

- **`record(Disposition)`** — the single write path to the provenance log. Append a
  dated decision-provenance entry for a recordable action (O002-R002).
- **`history(target) -> [ProvenanceEntry]`** — the dated change history of a target,
  decision-provenance included (O002-R002).
- **`recency(concept) -> Recency`** — when the concept last *meaningfully* changed
  (O002-R001).
- **`currency(concept) -> CurrencyStatus`** — `{ recency, drift_status }`, the observable
  half of the O002 proxy; `drift_status` is materialized from C004, not computed here.

## Data model

**The provenance log** — the one artifact C005 owns. Per ADR004 it is **plain-text,
in-corpus, append-only, and scoped** so history travels with the material rather than
depending on a central store (preserves O006 portability).

- **Physical scoping (component-internal choice):** one newline-delimited log file
  **per corpus directory**, co-located with the concept files that directory holds
  (e.g. `<dir>/.provenance.jsonl`). One entry per line; the file is only ever appended
  to. A slice of the corpus — any subtree — therefore carries the complete history of
  everything inside it, and a `git` diff of a mutation touches the concept file and its
  sibling log together. No cross-directory index is required to read a slice's history;
  a global view is a concatenation, computed on demand, never a stored second copy.
- **Entry (`ProvenanceEntry`):** the persisted form of a recorded `Disposition` =
  `{ target, outcome, actor, envelope_version, time, detail }`
  (shared types; `actor ∈ {wiki_auto, steward}`,
  `outcome ∈ {integrated, held_aside, merged, deprecated, superseded, relocated, flagged,
  declared, revised}` — the last two are policy-anchor declaration/revision from C001/C002),
  plus the entry's stable append position. Entries are immutable once written; a
  correction is a *new* entry, never an edit — append-only has no update or delete.
- **`CurrencyStatus` = `{ recency, drift_status }`** (shared type). `recency` is a
  `Recency` (last-meaningful-change time + which source it was read from);
  `drift_status` is a `DriftStatus` (`unchanged | drifted | unreachable`) **materialized
  from C004**, carried with an *as-of* time so consumers know how fresh the freshness is.
  Neither field is a stored artifact of C005's own — both are derived (see Behavior).

## Behavior

**`record` — the single write path.** Callers **report** a completed `Disposition`;
C005 appends it to the log file scoped to the target's directory. This is the *only*
mutation of the provenance store — there is **no shared mutable state**, and no caller
writes a log file itself. Called by **C008 on every corpus mutation** (as the atomic
concept-file-plus-provenance commit — ADR001) and by **C007, C011, C012, C001, C002**
on any other recordable action (hold-aside, retirement decision, scope flag, charter
revision, escalation resolution). `record` appends and returns; it does not evaluate,
route, or diff anything.

**`history`** replays the entries for a target (a concept, a directory, or the corpus)
in append order — the dated change history with decision-provenance for each change.
Read directly off the scoped log file(s), no recomputation.

**`recency`** reports the concept's last *meaningful* change. Two sources exist by
design (the deliberate, cheap redundancy of ADR004): the **inline recency stamp** C008
writes in the concept file, and the **provenance log**. The **inline stamp is
authoritative** for the value — C008 alone knows which mutations are *meaningful* versus
mechanical, and it stamps only the meaningful ones. The log is the co-located
cross-check and fallback: if the stamp is missing or the two disagree, C005 returns the
log-derived time and the disagreement is itself a currency signal. `recency` reads; it
never writes the stamp (that is C008's inline field, not C005's store).

**`currency`** returns `{ recency, drift_status }`. `recency` as above; `drift_status`
is **materialized from `C004.check_drift`** — C005 does **not** diff sources itself, and
does not store drift inline (ADR004: a source change must not force a concept-file
write).

- **Drift-refresh cadence (component-internal choice):** `check_drift` reaches a live
  locator (network — slow, and legitimately `unreachable`), so it is **never on the
  `currency` read path**. C005 runs a **periodic background sweep**, aligned with C011's
  retirement-review cadence (the principal consumer of drift), calling `C004.check_drift`
  per cited concept and caching the result with its as-of time. `currency` returns the
  last-materialized value; a read is always cheap and non-blocking. *Rationale:* the read
  path stays fast and offline-safe, drift-recompute cost stays bounded by a schedule
  rather than by read volume, and the as-of time keeps the staleness of the drift signal
  itself legible.

## Relationships

Matches [INTERFACES.md → Dependency direction](../INTERFACES.md#dependency-direction-no-cycles) exactly:

- **Depends on:** **C004** only — `check_drift` (materialize drift), `resolve` is *not*
  called here. C005 sits above C004, below the mutators.
- **Driven by (report to `record`):** all mutators — **C008** (every mutation),
  **C007, C011, C012, C001, C002** (recordable actions). These are *inbound* intent
  calls; C005 calls none of them back.
- **Read by:** **C011** — `currency` for staleness/drift retirement signals; and any
  **consumer** — `history` / `recency` / `currency` for the currency picture.
- **Single writer** of the provenance store; introduces **no cycle** (only downward
  edge is to C004).

## Decisions

- **[ADR004](../drs/ADR004-machine-readable-signal-placement.md)** governs this
  component. The log is **portable and scoped** (per-directory, plain-text, in-corpus —
  not a central DB), so history is portable at every layer with the material it
  describes. **Inline signals** (per-claim grounding + review tier, per-concept recency
  stamp, deprecation markers, reason-annotated cross-links) live in the **concept file
  written by C008**, *not* here. **Drift status is materialized here but computed by
  C004**, and is *never* stored inline (a source change must not churn a concept file).

## Success criteria

Tied to the **O002 proxy** — *proportion of concepts whose recorded recency and drift
status are up to date relative to their sources*:

- Every corpus mutation yields exactly one provenance entry (via C008's atomic commit),
  so `history` is complete and `recency` is derivable for every concept (O002-R001/R002).
- `currency` reflects each cited concept's drift within one refresh cycle of the source
  actually changing — the lag the O002 proxy measures — with an as-of time exposing that
  cycle's bound.
- The recency stamp and the log agree for concepts under C008's authorship; a
  disagreement is surfaced, not hidden.
- **Portability holds:** any corpus slice, moved or checked out on its own, carries its
  own complete history in its scoped log files with no external store or tooling
  required (O006) — verified by reading history off a detached subtree.
