# C003 - Integration Authoring

Turns a triaged item plus steward intent into corpus content: composes the concept, attaches its originating source link, and establishes contextual cross-links to related concepts with the reason for each link stated. When the item overlaps an existing concept, executes the steward-chosen reconciliation rather than authoring blindly. The steward supplies intent; this component performs the authoring.

## Behavior

This component authors concepts following the OKF conventions owned by C004 - OKF Conformance, and its writes are checked by C004's validator; it reasons about concepts and links, not the raw file format. Every authored or updated concept records a change event through C007 - Currency Tracking (which materializes the concept's `timestamp` and the scope's `log.md`), and its source link resolves to the captured-source snapshot the Inbox froze (C001), not only to a mutable external reference.

**New concept (no overlap).** Compose the concept from the item body and steward intent, attach the source link, create contextual cross-links to related concepts with the reason for each stated, and register the concept and its links with C005 - Index & Navigation so it becomes discoverable.

**Merge (item overlaps an existing concept).** C002 - Scope Triage surfaces the overlapping concept(s) with the merge direction. This component does not guess how to combine them — it executes the reconciliation the steward chose (see [ADR004 - Merge Reconciliation Steering](../drs/ADR004-merge-reconciliation-steering.md)):

- **Supersede** — author the replacement into the target concept and mark the prior statement deprecated in place, with a reason and a pointer to the replacement, through C008 - Lifecycle & Retirement. Inbound links to the prior statement are resolved first (via C005) so traversal never dead-ends.
- **Fold-in** — incorporate the material into the existing concept as a coexisting statement; nothing is deprecated.
- **Correct** — replace an erroneous or stale statement in place; the prior text is preserved in the concept's change history (C007), so what it used to say stays recoverable.

No reconciliation path hard-deletes the prior statement.

## Edge cases

- **Ambiguous or absent reconciliation choice**: the component does not author a merge on a guess — it returns to the steward for the reconciliation direction. Merge is never fire-and-forget.
- **Multiple overlapping concepts**: all are surfaced; the steward chooses a reconciliation per target (or directs a single target), and the component authors accordingly.
- **Overlap detected but the steward directs "keep as new"**: authored as a fresh concept, cross-linked to the near-neighbor with the reason stated, with no merge — the overlap becomes a link, not a reconciliation.
- **Originating source unknown**: the concept is still authored, with the captured-source snapshot as its resolvable source; a missing external reference is recorded as such rather than blocking authoring, consistent with C001 - Inbox.

## Relationships

- **C002 - Scope Triage**: supplies the triaged item, the steward's keep or merge direction, and — for a merge — the set of existing concepts the item overlaps that the reconciliation acts against.
- **C004 - OKF Conformance**: authors following the OKF conventions C004 owns, with its writes checked by C004's validator, rather than routing content through a runtime adapter.
- **C005 - Index & Navigation**: registers new concepts and their links, and resolves inbound links before a supersession relocates or deprecates a statement.
- **C007 - Currency Tracking**: records the change event whenever a concept is authored or updated, including each reconciliation.
- **C008 - Lifecycle & Retirement**: marks a superseded statement deprecated, with a reason and a replacement pointer, during a supersede reconciliation.

## Success criteria

- Every authored statement carries a source link that resolves to its captured-source snapshot (O001-R001), and every cross-link records the reason it exists (O003-R002).
- No merge is authored without an explicit steward reconciliation choice; an ambiguous or absent choice returns to the steward rather than being guessed.
- No reconciliation path silently discards the prior statement: supersede deprecates it with a pointer, correct preserves it in change history, fold-in retains both.
- After a supersede, no inbound link to the prior statement dangles.

## See Also

### Architectural Decision Records

- [ADR004 - Merge Reconciliation Steering](../drs/ADR004-merge-reconciliation-steering.md)
- [ADR007 - Conformance as Conventions and Validation](../drs/ADR007-conformance-conventions-and-validation.md)

### Change Records

- [CR002 - Steward-Attention Interactions](../../crs/CR002-steward-attention-interactions.md)
- [CR005 - Conformance Boundary Re-scope](../../crs/CR005-conformance-boundary-rescope.md)
