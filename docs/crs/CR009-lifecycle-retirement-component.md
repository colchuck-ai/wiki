# CR009 - Lifecycle & Retirement Component

Promoted **C008 - Lifecycle & Retirement** from an inline (Tier 0) statement on the engineering architecture document to its own Tier-1 component document, specifying its retirement approach, provenance-purge behavior, edge cases, and success criteria. Scoped to C008 and the engineering architecture document.

## Change

- **C008 - Lifecycle & Retirement**: promoted Tier 0 → Tier 1. Added a data model (deprecation and provenance-purged markers as in-corpus OKF markers), behavior (deprecate/supersede with inbound-link resolution; retirement review from measurable signals per PDR003; the provenance-purge cascade per ADR013), edge cases, relationships, and success criteria.
- **Engineering architecture document**: the C008 block collapses from an inline statement with a **Relationships** sub-list to reference form (one-liner plus a See link), with the relationships moved onto the new component document.

Before, C008 was the only standing capability left inline — a responsibility line and a relationships list, with no data model, no retirement method, no edge cases, and no success criteria — even though [CR008](CR008-delivery-form-and-standing-capabilities.md) promoted its peers C002, C005, C006, and C009. Its retirement input ("no longer relied upon") was asserted and never operationalized. After, C008 is a fully specified Tier-1 component whose retirement review runs on measurable signals.

## Rationale

C008 sits on the supersede path (ADR004) and is a named check of the Lint operation (ADR010), yet was the least-specified node in the tree. Finishing the promotion CR008 began — and giving retirement a measurable basis via PDR003 rather than an unmeasurable "relied upon" — closes that gap without changing any requirement.

## Affects

- Engineering: new C008 - Lifecycle & Retirement component document; the architecture document's C008 block becomes a reference. No requirement or outcome text changes. Retirement basis captured by [PDR003 - Reliance Is Steward-Supplied](../product/drs/PDR003-reliance-steward-supplied.md); provenance-purge behavior captured by [ADR013 - Provenance-Purge Cascade](../engineering/drs/ADR013-provenance-purge-cascade.md).
