# ADR013 - Provenance-Purge Cascade

When an explicit purge removes the captured source of an item that was **already integrated**, the concept(s) derived from it are flagged **provenance-purged** through C008 - Lifecycle & Retirement — so a consumer sees the claim is no longer source-backed, rather than the concept silently continuing to assert an unverifiable statement.

## Context

C001 - Inbox defines purge as the sole exception to retention, for material that must not be kept (secrets, personal data). But the captured snapshot is an integrated concept's **resolvable source** (C003 - Integration Authoring) and C006 - Source Revalidation's **baseline**. Purging the source of an already-integrated item silently makes that concept violate O001-R003 (captured-source retention) and strands C006 with no baseline — the exact O001-RSK002 failure the retention design exists to prevent. The interaction between purge and an already-integrated concept was unspecified.

## Options

- **Silent purge, concept unchanged** (rejected): the concept keeps asserting a claim whose captured provenance is gone, invisibly unverifiable — a hidden O001-R003 violation.
- **Block purge of an integrated item's source** (rejected): purge exists precisely for material that must not be retained; blocking it defeats the exception's reason to exist.
- **Cascade a provenance-purged flag to dependent concepts** *(chosen)*: allow the purge, and mark each derived concept provenance-purged, with a reason, so the loss of source backing is visible.

## Decision

Adopt the **provenance-purge cascade**. On a purge of an item whose `status` is `integrated`, C001 resolves the concept(s) it fed (via the `concept` backref) and C008 marks each **provenance-purged** — a lifecycle state carrying a reason, surfaced to consumers and to Lint's retirement and revalidation checks — while leaving the concept's curated content in place for the steward to decide whether to rewrite it from another source or retire it. C006 treats a provenance-purged concept as having no baseline (no drift reported) rather than erroring.

## Consequences

- Enabling: purge stays available for sensitive material without silently producing unverifiable claims; the provenance loss is explicit and consumer-visible.
- Enabling: reuses C008 lifecycle marking and the existing `concept` backref — no new store.
- Cost: a provenance-purged concept is a known, visible O001-R003 exception for that concept — the accepted tradeoff of honoring a purge; the steward can rewrite it from another source or retire it.
- Cost: adds a lifecycle state that C008 and consumers must understand, and a C006 branch for the no-baseline case.
