# CR002 - Citation pruning on merge reconciliation

Updated C003 - Integration Authoring's merge behavior: the `# Citations` block is now recomputed from the reconciled body on every merge, dropping any citation no longer backed by a claim in the body, instead of only ever unioning citations in.

## Change

Before: C003's document described a multi-source merge as "unioning" citations — the block only ever gained entries as sources were added, with no stated behavior for a merge that corrects an existing claim rather than adding beside it. A citation could remain listed after the claim it backed was rewritten or removed during reconciliation.

After: every merge recomputes the `# Citations` block from the reconciled body's inline markers — a citation with no remaining inline marker pointing to it is dropped, and the rest renumbered contiguously. This matches how C003 already composes citations for a `keep-new` concept (derived from what's actually written, nothing carried forward unreferenced). C003's document (Data model → Citation, Behavior → "Cite what it writes"/merge, Edge cases, Success criteria) is updated accordingly.

## Rationale

Checking CRUD coverage end-to-end surfaced the gap: "update an existing concept via new source material" is C003's merge path, and a *correcting* merge (new material replaces a claim rather than supplementing it) is a real instance of that, not an edge case outside the model. Leaving stale citations in place risked a `# Citations` block a reader couldn't use to tell current provenance from superseded provenance — exactly the low-signal drift O003 - Sustained signal exists to prevent. See [ADR010](../engineering/drs/ADR010-citation-pruning-on-merge.md) for the full option analysis, including why "accumulate forever" and "steward-signaled pruning" were both rejected.

## Affects

- **C003 - Integration Authoring** — the modified element; its Data model, Behavior, Edge cases, and Success criteria sections are updated.
- **O001-R001 - Cited claims** — served more strictly: every claim in the live document now resolves to a citation *and* every citation resolves to a claim, not just the former.
- **ADR010** — records the decision and alternatives considered.
