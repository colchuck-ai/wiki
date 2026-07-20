# C011 — Lifecycle & Retirement

**Role.** The retirement/relocation **decision-maker** for the *maintain* stage: it
surfaces stale, superseded, or mislocated concepts from measurable signals and drives
their marking, relocation, and link repair — mutating the corpus **only** by driving
C008 intent verbs, never by editing files. It owns the *decision*, not the write.

## Capabilities

Two, both expressed through the frozen § C011 surface (see
[INTERFACES.md](../INTERFACES.md#c011-lifecycle--retirement--the-mutating-maintainer)):

- **Surface candidates** — `review() -> [RetirementCandidate]`. Periodic (scheduled) or
  on-demand (steward). Sweeps consumer-facing concepts and flags those tripping the
  signal set below. This is the "surface" half of assisted upkeep (O004-R003) and the
  retirement review of O003-R002.
- **Drive the disposition** — for each candidate, *within the envelope* (`C002.evaluate`),
  drive exactly one of `C008.deprecate` / `C008.relocate` / `C008.repair_links`; escalate
  the calls it is not authorized to make via `C002.escalate`. It owns the two
  referential-integrity-sensitive moves — **retire and relocate** (O005-R003).

## Data model

**Stateless — a decision module, no persistent store.** Every durable fact it acts on is
read from another owner and every change it makes is committed by C008. It reads:

- `C005.currency(concept) -> CurrencyStatus` — `{ recency, drift_status }`: the staleness
  and source-drift signals (drift itself computed by C004, materialized by C005).
- **In-corpus supersession markers** — machine-readable retirement/supersession flags
  already on concepts (written earlier via `C008.flag`), read as part of the corpus.
- `C009.index()` to enumerate the concept set, and `C009.inbound_links(concept)` before any
  retire/relocate.

`RetirementCandidate` is **C011-local** (not a shared type): roughly
`{ concept, signals:[stale|drifted|superseded], proposed: deprecate|relocate|repair_links,
reason, successor? }` — a transient decision record produced by `review`, not stored.

## Behavior

**`review` — signal set.** For each consumer-facing concept, gather three signals and emit
a candidate if any fires:

1. **Staleness** — `currency.recency` past the freshness threshold with no meaningful change.
2. **Source drift** — `currency.drift_status ∈ { drifted, unreachable }`; the grounding
   source moved or vanished under the concept.
3. **Supersession** — an in-corpus supersession marker, or a newer concept that covers the
   same material (a candidate successor exists).

**Per-candidate disposition — decide, then drive.** C011 forms a `ProposedAction` and asks
`C002.evaluate`. If the decision is `act`, it drives the matching verb; if `escalate`, it
posts to `C002.escalate` with the signal evidence for the steward's judgment of what is
still relied upon (O003-R002). The decision rules:

- **Supersede → `C008.deprecate(concept, reason, successor)`.** A successor exists (from a
  supersession marker or an overlapping newer concept). Modeled as deprecate-*with*-successor
  — there is no separate `supersede` verb on the frozen C008 surface. Then drive
  `C008.repair_links(concept, successor)` so inbound links follow the replacement.
- **Deprecate → `C008.deprecate(concept, reason)` (no successor).** Obsolete or drifted with
  no replacement and no longer relied upon. The file is **marked in place, never deleted**,
  so inbound links still resolve to the (now-deprecated) concept — no repair needed and none
  dangles.
- **Relocate → `C008.relocate(concept, new_location)`.** Content is still valid and in-scope
  but sits at the wrong canonical path (OKF naming / reorganization). The move breaks inbound
  targets, so it is a referential-integrity move: `C008.relocate` triggers `repair_links` so
  inbound links follow to the new path (O005-R003 / RSK002).

**Referential-integrity ordering (hard rule).** Before *any* retire or relocate that could
strand references, C011 reads `C009.inbound_links(concept)` first, so the retarget set is
known and nothing is left dangling (O005-R003). Because deprecation marks in place rather
than deletes, plain deprecation never creates dangling links; relocate and supersede-with-
retarget are the cases that must drive `repair_links`. If a needed retarget is ambiguous
(no clean successor for links that are relied upon), C011 does **not** guess — it escalates.

**Provenance.** C011 does not write the log; each driven verb commits its concept change and
its `C005.record` entry atomically inside C008. Recordable review-level decisions (e.g. an
escalation) are reported to `C005.record` per that interface.

## Relationships

Matches the § C011 / "Dependency direction" contract exactly — **C011 → C002, C005, C008, C009**:

- **Reads** `C005.currency` (staleness/drift) and in-corpus supersession markers; enumerates
  via `C009.index`.
- **Reads** `C009.inbound_links` before every retire/relocate.
- **Drives** `C008.deprecate` / `C008.relocate` / `C008.repair_links` — *intent* calls; C008
  owns and commits the edit. C011 is not a corpus writer.
- **Governed by** `C002.evaluate` (act-vs-escalate is decided *only* there) and escalates
  ambiguous / unauthorized calls via `C002.escalate`.
- Reports recordable dispositions to `C005.record`.

Nothing reaches past these surfaces; C011 embeds no envelope logic and holds no state.

## Decisions

Both settled **in-component**, no ADR (consolidates the retired old ADR009/ADR012):

- **Deprecate-vs-delete → always mark in place.** Retirement is deprecation-over-deletion:
  the concept file is marked deprecated/superseded with a reason (and successor where one
  exists), never removed. Genuine purge is a separate, explicit, out-of-band act — not a
  C011 disposition. *Rationale:* keeps autonomy reversible (PDR003) and keeps inbound links
  resolving, so retirement cannot itself create dangling references.
- **Repair-to-successor.** On supersession and relocation, inbound links are retargeted to
  the successor / new path via `C008.repair_links`, always after reading
  `C009.inbound_links`. *Rationale:* O005-R003 requires inbound links be surfaced and
  resolved before a move, not left dangling.
- **Supersede = deprecate-with-successor** (internal): there is no distinct `supersede` verb
  on the frozen C008 surface, so C011 expresses supersession as `deprecate(…, successor)`.

## Success criteria

Tied to the O003 proxy — *proportion of consumer-facing concepts flagged (retirement,
supersession, duplication, significance) and not yet resolved*:

- **The flagged-and-unresolved rate falls.** `review` closes the loop from signal to
  disposition: candidates are acted on (within the envelope) or escalated and resolved by
  the steward — flags do not accumulate unresolved (counters O003-RSK001/RSK002).
- **Referential integrity is preserved across retire/relocate.** No *new* dangling links
  result from any C011-driven move: `C009.inbound_links` is read before every retire/relocate,
  and `C009.dangling()` shows no increase attributable to a disposition (O005-R003 / RSK002).
- **Nothing acts outside authorization.** Every disposition is either an envelope-permitted
  `act` or an `escalate`; C011 never marks or moves a concept without `C002.evaluate`.
- **Reversible.** Every retirement is an in-place marking, restorable — no deletions.
