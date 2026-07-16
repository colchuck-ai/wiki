# ADR009 - Retirement, relocation, and link-repair model

Give the steward a **free choice between two retirement mechanisms** — deprecate a concept in place (a marker plus an optional replacement pointer) or delete it outright — rather than deprecation-only; make deletion safe by having C008 **mechanically repair every inbound link** it orphans (unlink to plain text) while surfacing the affected concepts as **review candidates**, not silently resolved ones; and, because that repair capability exists anyway, give C008 a **relocate** operation that reuses it to retarget links instead of stripping them. Find retirement candidates through a self-contained three-signal sweep (staleness via C007, drift via C006, supersession via C008's own agent scan).

## Context

O003 - Sustained signal needs deprecation over deletion (O003-R001) and periodic retirement review (O003-R002), both mapped to C008 - Lifecycle & Retirement. The first draft of this component read O003-R001 and the architecture's "never dead-end the graph" principle as ruling out real file deletion entirely — every retirement would deprecate-in-place, forever, and relocation would be descoped because rewriting links inside concepts C008 doesn't own had no precedent anywhere in the system.

That draft was rejected in review: deprecate-only tombstones never leave the corpus, so the corpus can only grow — exactly the sprawl O003 - Sustained signal exists to prevent. O003-R001's actual wording ("rather than **only** removing it") already anticipated real deletion as a legitimate mechanism alongside deprecation, not a forbidden one; the never-dead-end principle had over-read it. O005-R003 - Referential integrity independently anticipates a real removal path: it requires inbound links be surfaced "before [a concept] is retired," which only matters if retirement can actually remove the thing being linked to.

Once deletion is real, two follow-on questions had to be resolved: what happens to the links that now point at nothing, and whether that same repair capability reopens relocation (rejected in an earlier pass of this same decision specifically because no component could safely rewrite another concept's links).

## Options

**Retirement mechanism — deprecate-only vs. steward's choice:**

- **Steward's free choice, no default (chosen)**: `deprecate` (marker in place) and `retire` (deletes the file) both exist as independent operations; the steward picks per item. Some retirements have a real replacement worth redirecting to (deprecate fits); others are just obsolete (retire fits, and actually shrinks the corpus).
- **Deprecate-only (the rejected first draft)**: never delete. Simplest mechanically, but guarantees monotonic corpus growth — directly working against O003's own outcome.
- **Delete-by-default, deprecate only for true supersession**: forces a lean toward deletion even for ambiguous cases. Rejected as an unnecessary default where the steward is already the adjudicator for every other admission and removal decision in this architecture (recommend-and-confirm, never-automatic) — a default here would be the one place the tool nudges instead of asking.

**What happens to links left dangling by a deletion:**

- **Auto-repair: mechanically unlink to plain text, and surface the referrers as review candidates (chosen)**: `retire` reads `C005.inbound_links`, rewrites `[title](path)` to plain `title` in every referrer (stripping only the link syntax, authoring nothing), and returns those referrers as a distinct `review_candidates` list — because a referrer's *prose* may have depended on the retired concept's content, not just its link, and that is a semantic question C008 cannot answer mechanically. Repair fixes the syntax; review candidacy is the steward's follow-up, not an automatic second edit.
- **Leave dangling, rely on periodic dangling_links sweeps**: matches "detect, don't repair" precedent elsewhere (C005, C006, C010), but leaves the corpus in a broken-reference state indefinitely between sweeps, for a repair that is entirely mechanical and safe to do immediately.
- **Block retirement on unresolved inbound links**: forces the steward to manually fix every referrer before retiring anything, contradicting "intent in, work out" — every other lifecycle operation in this architecture performs the work itself once directed.

**Whether relocation is in scope, now that link-rewriting is accepted:**

- **Reinstate relocation (chosen)**: `relocate(concept_id, new_path)` moves the file and retargets every inbound link to the new path, using the same mechanism `retire` uses to strip them. The objection that blocked this the first time — no component may rewrite links it doesn't own — is exactly what `retire`'s auto-repair now requires anyway, so paying that cost once for two operations is cheaper than paying it for one and awkwardly working around it for the other (author-elsewhere-plus-deprecate-original).
- **Keep relocation descoped**: consistent with the original ADR004/ADR005 self-contained-writer precedent, but now an inconsistent one — C008 already rewrites other concepts' links for `retire`; refusing to do the strictly simpler retarget for `relocate` buys no isolation benefit anymore.

## Decision

C008 exposes three steward-directed mutations and one read-only sweep. `deprecate(concept_id, reason, [replacement_concept_id])` marks a concept in place, exactly as before — never deletes, replacement pointer optional. `retire(concept_id)` deletes the concept's file outright; before/as part of deletion it reads `C005.inbound_links`, mechanically unlinks the reference to plain text in every referring concept (through C004's link-syntax helper), logs an `Update` against each repaired referrer and a new `Retirement` kind against the retired concept's own directory log, and returns those referrers as `review_candidates` for the steward's separate judgment about whether their content still holds up without what they cited. `relocate(concept_id, new_path)` moves the file and retargets (rather than strips) every inbound link to the new path, using the same repair machinery. `retirement_candidates(scope?)` remains a self-contained sweep across staleness (C007), drift (C006), and supersession (C008's own agent scan) — no dependency on C002 or C005 for discovery, no hard-coded thresholds.

This makes C008 the one component besides C003 that writes into a concept it isn't itself acting on — but only ever a mechanical link-syntax edit (strip or retarget), never authored prose, and always logged through C007 like any other content change.

## Consequences

- The corpus can actually shrink. `retire` is a real removal path, satisfying O003-RSK001 (accretion without removal) in a way deprecate-only tombstones cannot.
- C003's "only component that writes concepts into the corpus" claim is corrected: C003 remains the only component that *authors* concept content; C008 additionally performs narrow, mechanical link-repair edits as a side effect of `retire`/`relocate`, never composing new prose.
- C007's closed change-kind vocabulary grows to `{ Creation, Update, Deprecation, Retirement }`. A `Retirement` log entry names the concept by title without a working link, since the file it would point to no longer exists — a deliberate, singular exception to "log entries link the concept."
- `retire` and `relocate` never leave a dangling link behind as an end state — `C005.dangling_links` should, in steady state, reflect only genuinely orphaned or forward-referenced links, not ones caused by C008.
- Mechanical repair is not semantic resolution. `review_candidates` exists precisely because unlinking a reference doesn't tell you whether the referrer's claim still stands — that stays the steward's call, surfaced rather than silently assumed fine.
- `relocate` needs no `review_candidates` output: content is unchanged, only the path is, so there's no semantic-dependency question the way there is for `retire`.
- The supersession scan's cost still scales with corpus size, the same acceptable tradeoff C002's and C010's agent passes already make in the low-volume, single-steward envelope.

## Affected elements

- **C008 - Lifecycle & Retirement** — the component this decision defines; backlinks here.
- **C003 - Integration Authoring** — its "only writer of concept content" claim is narrowed to "only author of concept content"; backlinks here.
- **C005 - Index & Navigation** — `inbound_links` becomes load-bearing for `retire`/`relocate` (not merely informational), and its dangling-link edge cases are corrected; backlinks here.
- **C007 - Currency Tracking** — gains the `Retirement` kind and a relocation log-scoping edge case; backlinks here.
- **C006 - Source Revalidation** — `revalidate_all` consumed as C008's drift signal (no change to C006's own contract).
- **C010 - Charter** — `reconcile` candidates may be directed to C008 for `deprecate` or `retire` (no change to C010's own contract).
