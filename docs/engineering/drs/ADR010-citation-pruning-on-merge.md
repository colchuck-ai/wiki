# ADR010 - Citation pruning on merge reconciliation

When C003 reconciles a merge, the `# Citations` block always reflects exactly the sources currently backing a claim in the reconciled body — a citation with no remaining inline marker pointing to it is dropped and the rest renumbered contiguously, rather than accumulating indefinitely.

## Context

ADR005 settled multi-source merge citation granularity — concept-level by default, inline per-claim once a merge fuses more than one source — but only for the case where new material *adds beside* an existing claim. It never addressed what happens when new material *corrects* an existing claim: does the superseded citation stay listed even though no claim in the body still relies on it?

This surfaced while checking CRUD coverage end-to-end: "update an existing concept with new source material" is C003's merge path, and a correcting merge is a real instance of that — a table's column was renamed, a fact changed, and the new source's claim replaces the old one rather than sitting alongside it.

## Options

- **Prune to current claims (chosen)**: the `# Citations` block is recomputed from the reconciled body's inline markers on every merge — a citation nothing in the body still points to is removed, and the rest renumbered contiguously. Keeps the block exactly as trustworthy as any freshly-authored concept's, and mirrors what C003 already does for `keep-new` (compose the citation block from what's actually written, not carry forward anything unreferenced).
- **Accumulate forever**: never remove a citation once added, even after its claim is corrected away. Preserves a directly-addressable "here's what we used to think and why" in the live document, but produces a `# Citations` block a reader cannot use to tell current provenance from stale — exactly the low-signal drift O003 - Sustained signal exists to prevent, and it grows without bound across repeated corrections.
- **Steward-signaled pruning only**: prune only when the steward's merge note explicitly says "replace" rather than "add." Rejected — the note is freeform prose the steward writes for C003's reconciliation, not a structured flag; keying pruning off parsing it for intent adds a fragile branch for no benefit over just deriving citations from what the reconciled body actually cites.

## Decision

C003 recomputes the `# Citations` block as part of every merge reconciliation, exactly as it already does when composing a `keep-new` concept from scratch: every citation numbered in the block corresponds to at least one inline marker still present in the body; nothing else survives. A citation for a claim the reconciliation dropped or replaced is dropped along with it, and the remaining citations renumber contiguously.

## Consequences

- A correcting merge leaves no trace of the superseded citation in the live document. Nothing is actually lost — the prior claim, its citation, and the reconciliation that replaced it remain fully recoverable via `git log`/`git show` on the concept, and the event itself is dated in C007's `history(concept_id)` — but a reader of the current document sees only what's currently true and currently cited.
- Citation renumbering must be stable and deterministic (contiguous, in body order), the same determinism precedent C004 already applies to frontmatter key ordering, so re-authoring after a merge produces a minimal diff.
- No new interface surface. This refines behavior already inside `author`'s merge path (ADR005); C003's contract otherwise stands as specced.
- O001-R001 (cited claims) is served more strictly by this decision: every claim in the *live* document resolves to a citation, and every citation resolves to a claim — not just the former.

## Affected elements

- **C003 - Integration Authoring** — the modified element; backlinks here. See also [CR002](../../crs/CR002-citation-pruning-on-merge.md) for the concrete before/after.
- **ADR005 - Integration authoring, provenance, and merge model** — this decision extends, rather than revises, the merge model it settled; no contradiction, just a case ADR005 left open.
