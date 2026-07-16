# ADR012 - Repair-to-successor link model

Replace C008's two per-operation link-repair behaviors — retarget on `relocate`, strip-to-plain-text on `retire` — with a single **repair-to-successor** rule: when a concept leaves or moves, every inbound link is repaired to its *successor* — the new path (`move`), the replacement or merge target (`delete`-with-replacement, `merge`), or plain text when there is no successor (`delete` without replacement). `deprecate` needs no repair, since the file stays. This extends `delete` to accept an optional replacement and makes the `merge`-of-existing composition introduced in [ADR011](ADR011-concept-verb-surface.md) preserve navigability. Supersedes the split repair model in [ADR009](ADR009-retirement-relocation-and-link-repair-model.md).

> **Extended by [ADR014](ADR014-merge-composition-integrity.md):** the review-candidate suppression decided below for `merge` is given an explicit mechanism — a `content_preserved` parameter on `delete` that `merge` passes `true`, so the difference from a supersession `delete` is a truthful argument value rather than a merge-only code path. The repair-to-successor rule and the review-candidate-tracks-content-removal principle recorded here are unchanged.

## Context

[ADR009](ADR009-retirement-relocation-and-link-repair-model.md) established that C008 mechanically repairs the inbound links a lifecycle operation would orphan, and settled two behaviors: `retire` strips each referring link to plain text (the retired content is gone, so there is nowhere to point), and `relocate` retargets each to the new path (the content moved, so the link should follow). It did not contemplate a removal that has a *successor concept*, because at the time no such removal existed — [ADR005](ADR005-integration-authoring-provenance-and-merge.md)'s merge only ever folded an incoming task into a target it preserved in place, needing no repair.

[ADR011](ADR011-concept-verb-surface.md) changes that. `merge(from, into)` is now `revise(into, source=from)` then `delete(from, …)`: the `from` concept is genuinely removed, but its knowledge survives — it was just folded into `into`, which is exactly where `from`'s inbound links should now point. Under ADR009's rule, `delete` would strip those links to plain text, dead-ending readers who followed them even though the destination is known. ADR011 also introduces `delete(concept, replacement?)`, where a steward removing a superseded concept can name what replaces it.

So the repair question is no longer "retire or relocate"; it is **"does the departing concept have a successor to point at?"** — and both `move` and the new removals answer it differently than ADR009's two-case split can express. A second question rides along: `retire` surfaced `review_candidates` because a referrer's *prose* may have depended on content that is now gone. When the content demonstrably survives at a known successor, is that follow-up still warranted?

Constraints are ADR009's, unchanged: repair is only ever a mechanical link-syntax edit through C004's link helper, never authored prose; no operation leaves a dangling link as its end state; every repair is logged through C007; the envelope is low-volume and single-steward.

## Options

**How to repair links when a removal has a successor (`merge`, `delete`-with-replacement):**

- **Repair to successor — one rule parameterized by successor presence (chosen)**: unify `relocate`'s retarget and `retire`'s strip into a single rule. A departing or moving concept's inbound links are repaired to its successor: the new path (`move`), the replacement or merge target (`delete`-with-replacement, `merge`), or plain text when no successor is named (`delete` without replacement). `deprecate` needs no repair, because its file stays and the links still resolve. Navigability is preserved wherever a destination exists, and the three ADR009/ADR011 behaviors become three cases of one rule.
- **Keep strip-to-plain-text for every `delete` (ADR009's rule, unextended)**: simplest, but throws away a valid redirect on `merge` and `delete`-with-replacement — a reader following an inbound link lands on plain text instead of the concept that now holds the knowledge. Rejected: it works against O005 - Fast discovery precisely where the destination is known.
- **Deprecate the source instead of deleting it on `merge`**: leave `from` as a deprecated tombstone pointing at `into`. Rejected: it reintroduces the monotonic accretion ADR009 removed — a tombstone per merge — when a link redirect already gives readers the destination without keeping the dead file.

**Whether a redirecting removal still surfaces review candidates:**

- **Review candidates track content removal, not link repair (chosen)**: surface `review_candidates` when content is *removed* (`delete`, with or without a replacement — the replacement is a different concept not guaranteed to carry what a referrer depended on), and *not* for `move` or `merge`, where the depended-on content demonstrably survives at a known location (`move` leaves it untouched; `merge`'s own `revise` step folds it into the target the links now point at). This keeps ADR009's rationale — review candidacy asks "does the referrer's claim still hold now the cited content is gone?" — and answers it "not applicable" exactly when the content did not go anywhere.
- **Surface review candidates for every link repair**: uniform, but floods the steward with candidates on `move` and `merge` where nothing semantic changed, dulling the signal for the `delete` case that actually needs judgment. Rejected.

**Whether `delete`'s replacement must resolve:**

- **Require the replacement to name an existing concept (chosen)**: mirror `deprecate`'s existing rule — a replacement pointer must resolve, unlike an ordinary forward-referencing cross-link. A redirect target that does not exist would repair inbound links to nothing, reintroducing the dangling state the whole model forbids.
- **Tolerate a forward-referenced replacement**: consistent with OKF §5.3 for ordinary links, but a redirect destination is load-bearing, not advisory. Rejected for the same reason ADR009/C008 already refuse an unresolved `deprecate` replacement.

## Decision

Adopt the repair-to-successor rule. On any lifecycle operation, C008 repairs each inbound link to the departing or moving concept's successor: `move` retargets to the new path; `delete(concept, replacement)` and `merge(from, into)` retarget to the replacement/target concept; `delete(concept)` with no replacement strips to plain text; `deprecate` repairs nothing. `delete` accepts an optional `replacement`, which — like `deprecate`'s — must resolve to an existing concept or the call is refused. All repair remains a mechanical link-syntax edit through C004, never authored prose (ADR009's invariant, retained).

`review_candidates` are surfaced for `delete` (content removed) and not for `move` or `merge` (content preserved at a known successor). Every repair is logged through C007 exactly as ADR009 specified: `delete` records `Retirement` against the removed concept and `Update` against each repaired referrer; `move` records `Update` against the moved concept and each retargeted referrer; `merge` records `Update` against the surviving target (its `revise` step) plus `Retirement` against the folded-away concept and `Update` against each redirected referrer (its `delete` step).

This supersedes ADR009's split retarget/strip repair model. ADR009's other decisions stand: the steward's free choice of mechanism (now `deprecate` vs. `delete`, held apart as two verbs), mechanical-repair-only, no dangling end state, and the self-contained retirement-candidate sweep.

## Consequences

- Navigability survives every operation that has a destination: `merge` shrinks the corpus (no tombstone) while inbound links follow the knowledge into the target, serving O005 - Fast discovery and O003 - Sustained signal together rather than trading one for the other.
- The three behaviors collapse to one rule with a single question — "is there a successor?" — which is simpler to specify, test, and reason about than ADR009's per-operation cases, and it closes a latent hole: ADR009's `retire` would have stripped links even when a replacement existed, because it had no way to redirect them.
- `review_candidates` narrows to the case that needs it. `move` and `merge` produce none; only `delete` (content actually removed) prompts the steward to check whether referrers still hold up. The `merge` composition therefore suppresses the review-candidate surface its `delete` step would otherwise raise, because its `revise` step preserved the content.
- `delete` gains a resolve-or-refuse check on its optional replacement, matching `deprecate`; a non-resolving replacement is rejected rather than allowed to orphan links.
- C005 - Index & Navigation's `inbound_links` stays load-bearing for `move`, `delete`, and `merge` (each reads it to find what to repair), and `dangling_links` still reflects only genuinely orphaned or forward-referenced links in steady state — never one caused by a lifecycle operation.

## Affected elements

- **C008 - Lifecycle & Retirement** — the component this decision defines: `delete` gains an optional resolve-checked replacement, the repair rule unifies across `move`/`delete`/`merge`, and `review_candidates` narrows to `delete`. Backlinks here.
- **C003 - Integration Authoring** — the `merge` composition (ADR011) relies on `delete`-with-replacement to redirect the folded-away concept's inbound links; its `revise` step is why `merge` surfaces no review candidates. Backlinks here.
- **C005 - Index & Navigation** — `inbound_links` remains load-bearing for the redirecting removals; the dangling-link end-state guarantee is unchanged. Backlinks here.
- **C007 - Currency Tracking** — records the `Retirement`/`Update` fan-out for `delete`, `move`, and the composed `merge` as specified above (no new change-kind beyond ADR009's vocabulary). Backlinks here.
