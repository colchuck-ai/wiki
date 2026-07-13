# CR002 - Steward-Attention Interactions

Specified the two points in the curation loop where the steward's judgment is required beyond a one-word verdict — reconciling a **merge** against an overlapping concept, and **pivoting** a reference's purpose — and promoted the two components that own them (C003 - Integration Authoring and C010 - Scaffold) from inline statements to their own documents. Scoped to C002 - Scope Triage, C003 - Integration Authoring, C010 - Scaffold, and the engineering architecture document; captured by ADR004 and ADR005.

## Change

- **C003 - Integration Authoring**: promoted from Tier 0 (inline) to Tier 1 (own document). Added merge-reconciliation behavior — on an item that overlaps an existing concept, the steward chooses **supersede**, **fold-in**, or **correct** and the component executes it — plus edge cases, success criteria, and two new relationships (C007 - Currency Tracking for change events, C008 - Lifecycle & Retirement for supersession). Before, C003 described only composing a *new* concept with a source link and cross-links; the fate of an item that overlaps an existing concept was unspecified, so "merge" — a direction O007-R001 already names — had no defined behavior.
- **C002 - Scope Triage**: enriched (still Tier 0) to detect and surface the existing concept(s) an incoming item overlaps, so a merge direction can be steered against a concrete target rather than issued blind.
- **C010 - Scaffold**: promoted from Tier 0 to Tier 1. Added the guided fork-and-migrate flow — detect a scaffold edit that would change the immutable core purpose, refuse the in-place rewrite, and drive a new-bundle-plus-per-concept-migration — plus core-vs-periphery handling, edge cases, and success criteria. Before, PDR001 established the immutable-core decision but nothing detected or guided an attempted in-place core rewrite; it was a documented manual procedure.
- **Engineering architecture document**: collapsed the C003 and C010 inline blocks to reference form (one-liner plus a See link), moved their relationships onto the new component documents, and updated the C002 responsibility line.

## Rationale

Under the low-volume assumption and the steward's trust in the tool's authoring, keep and discard are fire-and-forget. That leaves exactly two moments that require the steward's judgment. A **merge**: the tool must not guess whether new material supersedes, coexists with, or corrects an existing concept, because that is an editorial decision about what the corpus asserts, not a mechanical one. A **purpose pivot**: an in-place core rewrite would silently invalidate past scope judgments and break the contract PDR001 protects. Specifying both closes the gap between the decisions already recorded (O006-R001 deprecation-with-pointer, O007-R001 integrate-by-intent, PDR001 immutable core telos) and the interaction a steward actually experiences.

## Affects

- Engineering: C003 - Integration Authoring and C010 - Scaffold (new component documents and behavior), C002 - Scope Triage (responsibility), and the architecture document's component blocks. Captured by [ADR004 - Merge Reconciliation Steering](../engineering/drs/ADR004-merge-reconciliation-steering.md) and [ADR005 - Guided Fork-and-Migrate](../engineering/drs/ADR005-guided-fork-and-migrate.md).
- Product: no requirement or outcome text changes — the two interactions realize requirements that already exist (O006-R001 deprecation over deletion, O007-R001 agent-authored integration) and enforce a product decision that already exists (PDR001 immutable core telos). ADR005 supersedes PDR001's *consequence* that no capability beyond documentation is required; the product decision itself is unchanged.
