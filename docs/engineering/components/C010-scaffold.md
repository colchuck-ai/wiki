# C010 - Scaffold

Lets the steward declare the reference's purpose and scope as an optional, persistent anchor that endures for the life of the reference; distinguishes the immutable core purpose from the revisable periphery; detects a scaffold edit that would change the core purpose and routes the steward to fork-a-new-reference-and-migrate rather than applying it in place; and supports periodically reconciling the graph against the declared scope to surface content that has drifted out of scope.

## Data model

The scaffold is a plain-text, version-controlled artifact that travels with the bundle like the rest of the corpus (O005). It holds two parts with different mutability:

- **Core purpose** — the statement of what the reference is for. Fixed for the life of the reference (see [PDR001 - Immutable Core Telos](../../product/drs/PDR001-immutable-core-telos.md)).
- **Periphery** — supporting boundaries and context. Revisable as understanding deepens.

The scaffold is optional. With none declared, there is no scope contract for triage to adjudicate against and no core-change detection applies.

## Behavior

**Declaration.** The steward articulates the purpose — from a single statement of intent to a fuller set of boundaries and context — and the tool maintains it thereafter. The steward never hand-edits the corpus to establish scope; they declare intent and the tool holds the anchor.

**Periphery refinement.** Edits that refine boundary detail or context are applied in place. The core purpose is untouched.

**Core-change detection and guided fork-and-migrate.** When a proposed edit would alter the core statement of purpose rather than the periphery, the tool does not apply it in place. It identifies the edit as a core-purpose change and initiates the guided flow (see [ADR005 - Guided Fork-and-Migrate](../drs/ADR005-guided-fork-and-migrate.md)):

1. Fork a new bundle with a fresh scaffold whose core purpose the steward declares.
2. Migrate still-in-scope concepts across one at a time, the steward adjudicating each, relying on O005 portability (plain-text, OKF-conformant, version-control-native concepts copy cleanly) — no bespoke migration machinery.
3. Leave the original reference's core purpose and scope contract unchanged.

**Scope reconciliation.** Periodically traverse the graph (via C005 - Index & Navigation) against the declared scope to surface content that has drifted out of scope, presenting it for the steward's judgment rather than acting automatically.

## Edge cases

- **Core-vs-periphery ambiguity**: when the tool cannot tell whether an edit refines the periphery or changes the core, it flags the edit as potentially core-changing and asks the steward to adjudicate. Per PDR001 the line can be fuzzy and is settled as an act of steward intent.
- **Item pending during a fork**: an item that motivated the pivot stays `pending` in the original inbox until the new bundle exists, then is captured as the new bundle's intake — it does not silently migrate.
- **Migrating a concept with inbound links from concepts left behind**: inbound links are resolved (via C005) before the concept is carried over, so neither the original nor the new bundle is left following a dangling reference.

## Relationships

- **C002 - Scope Triage**: provides the scope contract each incoming item is adjudicated against.
- **C005 - Index & Navigation**: traverses the graph during reconciliation to find drifted content, and resolves inbound links during a migration.

## Success criteria

- The declared core purpose is never rewritten in place: a core-changing edit is always detected and routed to fork-and-migrate, never applied to the existing scaffold (O008-R001, PDR001).
- Peripheral boundary and context can be refined without a fork.
- After a fork, the original reference's scope contract is unchanged, and each migrated concept remains OKF-conformant and source-linked in the new bundle.
- Scope reconciliation surfaces out-of-scope drift against the declared scope (O008-R003) rather than leaving it to persist unnoticed.

## See Also

### Architectural Decision Records

- [ADR005 - Guided Fork-and-Migrate](../drs/ADR005-guided-fork-and-migrate.md)

### Change Records

- [CR002 - Steward-Attention Interactions](../../crs/CR002-steward-attention-interactions.md)
