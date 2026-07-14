# CR011 - Revisable Core Purpose

Reversed the immutable-core-telos decision: the scaffold's core purpose may now be **revised in place**, triggering a Survey-driven reconciliation instead of a fork to a new bundle. Updated the product narrative, the engineering architecture constraint and principle, and C010 - Scaffold; superseded PDR001 and ADR005. Scoped to the product README, the engineering architecture document, the O008 outcome, and C010 - Scaffold. Captured by [PDR004 - Revisable Core Purpose](../product/drs/PDR004-revisable-core-purpose.md) and [ADR011 - In-Place Core-Purpose Change](../engineering/drs/ADR011-in-place-core-purpose-change.md).

## Change

- **Product README**: the narrative that "its core purpose is fixed — redefining that purpose constitutes a new reference … rather than an edit" is reworded — a core-purpose revision is a deliberate in-place act that triggers a Survey-driven reconciliation of the corpus against the new purpose. O008 backlinks to PDR004.
- **Engineering architecture document**: the constraint "a change of purpose is a new bundle plus a migration, not an in-place edit, so no component offers a 'rewrite purpose' operation" is reversed to an in-place revision plus Survey-driven reconciliation (PDR004, ADR011); the "scaffold is the single scope authority" principle drops "core purpose is fixed for the life of the reference"; the See Also references PDR004/ADR011.
- **C010 - Scaffold**: core-change detection no longer blocks and routes to fork-and-migrate; a core-affecting revision applies in place and initiates the reconciliation workflow (revise → Survey → plan → execute). The cross-bundle migration edge cases are removed. Behavior, edge cases, and success criteria updated.
- **Decision trail**: PDR004 supersedes [PDR001 - Immutable Core Telos](../product/drs/PDR001-immutable-core-telos.md); ADR011 supersedes [ADR005 - Guided Fork-and-Migrate](../engineering/drs/ADR005-guided-fork-and-migrate.md). ADR009's context reference to fork-and-migrate is reworded to "purpose-pivot reconciliation."

Before, a genuine pivot meant forking a new bundle and migrating concepts one at a time — which would partition a live graph across two bundles and require cross-bundle relationships OKF does not define. After, the pivot is an in-place revision reconciled through Survey, and cross-bundle relationships never arise; the prior whole-bundle state is preserved by version control.

## Rationale

Fork-and-migrate would have to split a connected graph across two bundles, needing cross-bundle links the format does not define, and cloning a whole bundle to preserve prior state duplicates version control. In-place revision plus Survey-driven reconciliation is lighter and reuses existing capabilities, at the deliberate cost of changing the scope contract from *durable* to *current-as-of-now*. See PDR004 for the decision and its consequences.

## Affects

- Product: the README core-purpose narrative; O008 backlink to PDR004.
- Engineering: the architecture document's Constraints and Principles and See Also; C010 - Scaffold (behavior, edge cases, success criteria); ADR009 context wording.
- Decision trail: PDR004 supersedes PDR001; ADR011 supersedes ADR005. Captured by [PDR004 - Revisable Core Purpose](../product/drs/PDR004-revisable-core-purpose.md) and [ADR011 - In-Place Core-Purpose Change](../engineering/drs/ADR011-in-place-core-purpose-change.md).
