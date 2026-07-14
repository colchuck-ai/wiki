# CR013 - Provenance-Purge Cascade

Specified that purging the captured source of an already-integrated item cascades a **provenance-purged** mark to the concept(s) it fed, so the loss of source backing is visible rather than silent. Scoped to C001 - Inbox and C008 - Lifecycle & Retirement (with a C006 - Source Revalidation branch). Captured by [ADR013 - Provenance-Purge Cascade](../engineering/drs/ADR013-provenance-purge-cascade.md).

## Change

- **C001 - Inbox**: the purge edge case is extended — on a purge of an item whose `status` is `integrated`, C001 resolves the concept(s) it fed (via the `concept` backref) and hands them to C008 to mark provenance-purged.
- **C008 - Lifecycle & Retirement**: gains a provenance-purged lifecycle marker, carrying a reason, applied to a concept whose captured source was purged.
- **C006 - Source Revalidation**: treats a provenance-purged concept as having no baseline (no drift reported) rather than erroring.

Before, C001's purge left the fate of an already-integrated concept unspecified — the concept silently kept asserting a claim whose captured provenance was gone, and revalidation lost its baseline (a hidden O001-R003 violation). After, the loss is explicit and consumer-visible, and the steward can rewrite the concept from another source or retire it.

## Rationale

Purge exists for material that must not be retained, so blocking it is not an option; but a silent purge produces an invisibly unverifiable claim. Cascading a visible provenance-purged mark keeps purge available while making the consequence explicit, reusing C008 lifecycle marking and the existing `concept` backref with no new store. See ADR013 for the decision.

## Affects

- Engineering: C001 - Inbox (purge edge case), C008 - Lifecycle & Retirement (provenance-purged state), C006 - Source Revalidation (no-baseline branch). No requirement or outcome text changes. Captured by [ADR013 - Provenance-Purge Cascade](../engineering/drs/ADR013-provenance-purge-cascade.md).
