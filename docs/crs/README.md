# Change Records (CR)

**The change-record log starts empty.**

A **Change Record** documents a change to an already-frozen intent element — a
requirement, a component contract, an interface, or a decision — after the tree
has been settled. It names *what changed*, *why*, and *what it affects*, so the
edit is traceable rather than silent.

There are no open CRs. The engineering tree was re-derived clean-room against the
rewritten product tree (2026-07); the change history that produced the old tree
(old CR001–CR008, CR010) is archived under
[`archive/docs-v2/crs/`](../../archive/docs-v2/crs/). Its still-relevant substance
was folded into the current components and ADRs during that rewrite, so it carries
forward as settled architecture, not as a running log:

- old CR001 → **ADR003** (citation two-reference model)
- old CR002 → **C008** `merge` citation pruning
- old CR004 → **C011** repair-to-successor link model
- old CR005 → **ADR001** per-verb atomicity (merge is one verb; the cross-component
  composition concern dissolves)
- old CR006 → **REQUIREMENT-MAP** O004-R003 shared ownership
- old CR007 → **C008** author-time grounding seam + **ADR004** signal placement
- old CR008 → **consciously dropped** (the Survey/Lint aggregator is old-tree
  gold-plating; see [DECOMPOSITION drop #1](../engineering/DECOMPOSITION.md#consciously-dropped))
- old CR010 → absorbed by **C009**'s compute-on-read freshness (as-of revision)

## When to add a CR

Add a numbered `CRNNN-short-slug.md` here when a change alters a frozen element
after the fact. Record the change, the rationale, and the affected elements, and
update the elements it points at so the tree and the log never disagree. A change
that only *creates* new elements (not yet frozen) needs no CR.
