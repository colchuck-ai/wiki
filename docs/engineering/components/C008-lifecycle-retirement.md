# C008 - Lifecycle & Retirement

Retires a concept — by deleting it outright (auto-repairing every inbound link) or by marking it deprecated or superseded in place with a reason and, where one exists, a replacement pointer — relocates a concept to a new path with its inbound links retargeted automatically, and surfaces retirement candidates from measurable signals (staleness, source drift, supersession) for the steward's judgment.

C008 is the pipeline's **mutation-of-existing-concepts** component, the counterpart to C003's mutation of new ones: where C003 turns admitted material into a concept, C008 turns the steward's retirement or relocation judgment into a change to one already in the corpus. It fulfills deprecation over deletion (O003-R001) and retirement review (O003-R002) outright. Deprecation and supersession are **one mechanism, not two**: a concept is deprecated with a reason, and carries a replacement pointer only when one exists — "superseded" is simply the case where that pointer is set. Retirement is a **free choice between two mechanisms**: `deprecate` marks in place and never deletes; `retire` deletes the file outright. Neither is a default — the steward picks per item, since only they know whether a real replacement makes a redirect worth keeping or the material is simply obsolete. Like C002, C008 both surfaces candidates for judgment and executes the steward's confirmed action.

**C008 is the one component besides C003 that writes into a concept it isn't itself acting on.** Deleting or moving a concept can orphan the links inside every concept that referenced it; C008 repairs those mechanically — stripping a dead link to plain text (`retire`) or retargeting it to the new path (`relocate`) — through C004's link-syntax helper, never composing new prose. This is a deliberate, narrow exception to C003 being the sole *author* of concept content, not a second authoring path. See [ADR009](../drs/ADR009-retirement-relocation-and-link-repair-model.md) for the full reasoning, including why an earlier draft of this component ruled out deletion and relocation entirely and why that was reversed.

## Data model

- **Deprecation marker.** A Wiki convention layered on OKF's tolerant extension-key and heading conventions, read and written through C004:
  - **`deprecated: true`** — a frontmatter boolean, the cheap discovery key (mirrors C010's `type: charter` as a semantic convention C004 neither registers nor constrains).
  - **`# Deprecation`** — a body section, required once `deprecated` is set: prose stating the reason, and, when a replacement exists, a reasoned cross-link to it (a bundle-relative link with the reason in the surrounding prose, per OKF §5.3 and C003's link convention) — never a raw frontmatter path, so the replacement pointer is a knowledge cross-link like any other.
- **Repaired-link result** (transient, not written to the corpus beyond the edit itself). One per referring concept `retire` or `relocate` mechanically edits: `{ concept-id, before, after }` — the link text before and after the edit, so the steward can see exactly what changed.
- **Review candidate** (transient). One per referring concept `retire` surfaces after repairing its link: `{ concept-id, retired-concept-id, rationale }` — a prompt to check whether that concept's own content still holds up without what it cited, distinct from the mechanical repair itself. `relocate` produces no review candidates — content is unchanged, only location is.
- **Retirement candidate** (transient, not written to the corpus). One per concept the periodic sweep flags: `{ concept-id, signals: [{ kind ∈ { staleness, drift, supersession }, rationale }], candidate-replacement? }`. A concept can carry more than one signal at once; they are not deduplicated into a single verdict.

## Interfaces

C008 exposes an in-process face. Every corpus read and write goes through C004. Two operations write beyond the concept named — see Behavior.

- **`deprecate(concept_id, reason, [replacement_concept_id]) → concept`** — writes the marker via C004. Calls `C005.reindex()` and `C007.record(concept_id, kind, reason)` after. Refuses `replacement_concept_id` that does not resolve to an existing concept, and refuses the charter concept outright. Never deletes anything and never edits another concept.
- **`retire(concept_id) → { removed_path, links_repaired[], review_candidates[] }`** — deletes the concept's file via C004. Reads `C005.inbound_links(concept_id)` first to find every referrer; for each, mechanically unlinks the reference to plain text through C004's link-syntax helper and calls `C007.record(referrer_id, kind=Update, ...)`. Logs the retirement itself via `C007.record(concept_id, kind=Retirement, reason)`. Calls `C005.reindex()` last. Returns the repaired links and, separately, the same referrers as `review_candidates`. Refuses the charter concept.
- **`relocate(concept_id, new_path) → concept`** — moves the concept's file to `new_path` via C004's path helpers. Reads `C005.inbound_links(concept_id)` and retargets each referrer's link to `new_path` (through C004's link-syntax helper), calling `C007.record(referrer_id, kind=Update, ...)` for each. Calls `C007.record(concept_id, kind=Update, ...)` for the moved concept itself and `C005.reindex()` last. Content is unchanged; only its path and the links pointing to it are. Produces no review candidates.
- **`deprecation_of(concept_id) → { reason, replacement-concept-id? } | none`** — reads the marker, `none` when the concept carries no `deprecated` flag.
- **`retirement_candidates(scope?) → candidate[]`** — the periodic sweep behind the product's **Survey** operation: reads staleness from `C007.recency`/`history`, drift from `C006.revalidate_all`, and runs its own agent scan through C004 for supersession — self-contained, no dependency on C002 or C005 for candidate discovery, the same posture [ADR004](../drs/ADR004-triage-disposition-model.md) set for C002. Excludes concepts already deprecated and excludes the charter. Read-only.

## Behavior

- **Deprecate in place; retire deletes; steward picks, no default.** `deprecate` never removes a file; `retire` always does. `retirement_candidates` recommends nothing about which mechanism to use — it surfaces signals, and the steward chooses `deprecate`, `retire`, or neither.
- **Mechanical repair only, never authored prose.** The one edit `retire`/`relocate` makes to a concept other than the one named is stripping or retargeting a link's syntax — through C004's link-syntax helper, which recognizes and rewrites a bundle-relative link without touching surrounding text. C008 never rewrites a sentence, drops a claim, or composes new prose in a concept it doesn't own.
- **Repair, then surface for review — don't assume repaired means resolved.** `retire` fixes the syntax (no dangling link survives) and separately returns `review_candidates`, because a referrer's prose may have depended on the retired concept's content in a way no mechanical edit can judge. Whether that referrer's claim still holds is the steward's call, not a second automatic fix.
- **Relocation carries no semantic risk, so no review surface.** Moving a concept changes nothing about its content — only `retire` (real removal) creates the "does the referrer still make sense" question `review_candidates` exists to flag.
- **Recommend, never retire or relocate automatically.** `retirement_candidates` only surfaces; only a direct `deprecate` or `retire` call changes anything, mirroring C002's recommend-and-confirm posture and the architecture's never-automatic principle applied to removal, not just admission.
- **Self-contained supersession scan.** The scan for "a newer concept covers this ground" is C008's own agent pass over the corpus through C004, not a call into C002's overlap machinery or C005's graph — consistent with the self-contained precedent ADR004 and ADR005 established.

## Edge cases

- **Deprecating with no replacement**: `reason` is required, `replacement_concept_id` is not; the `# Deprecation` section omits the replacement link entirely.
- **Deprecating an already-deprecated concept**: edits the reason and/or replacement in place; logged via `C007.record` as `Update`, not a second `Deprecation` event.
- **`replacement_concept_id` does not resolve**: refused outright — unlike an ordinary reasoned cross-link (which tolerates a forward reference per OKF §5.3), a replacement pointer must name something real.
- **Retiring a concept with no inbound links**: `links_repaired` and `review_candidates` are both empty; the deletion proceeds regardless.
- **Retiring a concept linked from more than one place in the same referrer**: each occurrence is repaired independently; the referrer appears once in `review_candidates` regardless of how many links to the retired concept it held.
- **Relocating a concept to a path that already holds a concept**: refused — `relocate` never overwrites an existing concept, mirroring C003's `keep-new` slug-collision handling.
- **The charter concept**: `deprecate` and `retire` both refuse it, and `retirement_candidates` never includes it — revising its purpose is `C010.revise_charter`, not a retirement.
- **A concept flagged by more than one signal** (e.g. stale and drifted): surfaced once, carrying both signal entries.
- **Retirement candidate already deprecated**: excluded from `retirement_candidates` output.

## Relationships

- **C004 - OKF Conformance**: sole path to corpus content — the deprecation-marker frontmatter/body convention, the file-delete and path-move helpers `retire`/`relocate` use, and the link-syntax helper both use to strip or retarget a referrer's link. C008 encodes no OKF structural literal.
- **C005 - Index & Navigation**: `inbound_links` is now load-bearing, not merely informational — `retire` and `relocate` both read it to find every concept they must mechanically edit. Both call `reindex` last. C005 applies no exclusion to deprecated concepts (they stay indexed, like the charter); a retired concept simply no longer appears once its file is gone.
- **C006 - Source Revalidation**: `retirement_candidates` reads `revalidate_all` and treats a `drifted` verdict as one retirement signal (O003-R002's "source drift"). C008 performs no fetch itself.
- **C007 - Currency Tracking**: `retirement_candidates` reads `recency`/`history` as the staleness signal. `deprecate` calls `record` with `kind = Deprecation` (fresh) or `Update` (edit). `retire` calls `record` with the new `kind = Retirement` against the retired concept, plus `kind = Update` against every repaired referrer. `relocate` calls `record` with `kind = Update` against both the moved concept and every retargeted referrer.
- **C010 - Charter**: excludes the charter from `retirement_candidates` and refuses both `deprecate` and `retire` on it. When `C010.reconcile()` surfaces an out-of-scope concept, the steward may direct C008 to `deprecate` or `retire` it — C010 detects scope drift, C008 executes.
- **C003 - Integration Authoring**: no runtime dependency. C008's mechanical link edits never touch a concept's authored prose, only a link's syntax, so C003's ownership of *authoring* content is undiminished — only its claim to being the sole *writer* of any kind is narrowed (see [ADR009](../drs/ADR009-retirement-relocation-and-link-repair-model.md)).
- **C002 - Triage**: no relationship — C002's duplication check runs against incoming material; C008's supersession scan runs against the existing corpus, a distinct pass over different material.
- **Boundary**: C008 owns marking a concept deprecated, deleting or relocating a concept, mechanically repairing the links that action orphans or retargets, and surfacing retirement candidates from staleness, drift, and supersession signals. It never authors new prose in a concept it isn't acting on, never judges whether a referrer's content still holds up (only surfaces the question), and never judges scope or duplication at intake (C002, C010).

## Success criteria

- **Steward's choice preserved**: no path picks `deprecate` or `retire` on the steward's behalf — `retirement_candidates` never mutates the corpus, and each of `deprecate`/`retire`/`relocate` only runs when called directly.
- **No dangling link survives a retirement or relocation**: after `retire` or `relocate` completes, `C005.dangling_links` contains no link that used to target the affected concept — verifiable by retiring or relocating a linked-to concept and re-running `dangling_links`.
- **Mechanical repair only**: a diff of any referrer C008 edits shows only the link's markup changed (text or target) — no other line differs.
- **Review surfaced, not silently resolved**: every referrer `retire` repairs also appears in `review_candidates` — verifiable by retiring a concept with at least one inbound link and confirming both lists name it.
- **Replacement resolves**: a `deprecate` call carrying `replacement_concept_id` only succeeds when that id resolves to an existing concept.
- **Charter excluded**: `retirement_candidates` never includes the charter concept; `deprecate` and `retire` both refuse it.
- **One-owner invariant preserved**: C008 contains no OKF structural literal — ADR001's grep-able check stays clean with C008 present.

## Notes

- The deprecation marker (`deprecated: true` plus a `# Deprecation` body section) is a Wiki convention recorded here rather than a separate ADR, the same treatment C007 gave its log-kind vocabulary — promote it if it becomes contested.
- The retirement/relocation/link-repair model — steward's free choice of mechanism, mechanical-repair-plus-review-candidates for deletion, and relocation's reinstatement once repair became an accepted capability — is captured with alternatives in [ADR009](../drs/ADR009-retirement-relocation-and-link-repair-model.md).
- Retirement review is surfaced through the product's **Survey** operation, alongside C006's `revalidate_all` and C010's `reconcile`.

## See Also

### Architectural Decision Records

- [ADR009 - Retirement, relocation, and link-repair model](../drs/ADR009-retirement-relocation-and-link-repair-model.md)
