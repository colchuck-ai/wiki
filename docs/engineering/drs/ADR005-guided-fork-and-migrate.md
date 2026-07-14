# ADR005 - Guided Fork-and-Migrate

> **Superseded by [ADR011 - In-Place Core-Purpose Change](ADR011-in-place-core-purpose-change.md).** With the core purpose now revisable in place ([PDR004](../../product/drs/PDR004-revisable-core-purpose.md)), a pivot no longer forks a new bundle and migrates concepts across it; C010 applies the revision in place and initiates a Survey-driven reconciliation, avoiding the cross-bundle relationships a fork would require. This record is preserved for the decision trail.

Realize the immutable-core-telos decision ([PDR001](../../product/drs/PDR001-immutable-core-telos.md)) as an active, tool-guided flow: C010 - Scaffold detects a scaffold edit that would change the core purpose, refuses to apply it in place, and drives a fork-a-new-reference-plus-per-concept-migration workflow — rather than leaving fork-and-migrate as a purely documented manual procedure.

## Context

PDR001 fixed the scaffold's core purpose for the life of the reference and modeled a genuine pivot as a new bundle plus a migration. It concluded that migration "needs no bespoke machinery — it falls out of O005 portability," and that "no new product capability is required beyond documenting the workflow (fork the bundle, carry over in-scope concepts, re-anchor the scaffold)."

Working through the steward interaction exposes a gap that documentation alone leaves open. A steward will naturally phrase a pivot as an in-place edit — "change the purpose to also cover X." If the tool applies scaffold edits at face value, it will rewrite the core purpose in place, silently invalidating every past triage (O008-R002) and reconciliation (O008-R003) judgment and breaking the scope contract PDR001 exists to protect ("if it isn't here, it's out of scope, not missing"). The architecture must decide whether the immutable-core rule is merely documented or actively enforced, and how a legitimate pivot is carried out when one is genuinely intended.

## Options

- **Documented manual workflow only** (PDR001's original stance): rely on the steward to know that a pivot means fork-and-migrate and to perform it by hand. No new capability, but nothing detects an in-place core rewrite, so the contract is one careless edit away from silently breaking.
- **Block-only enforcement**: detect and refuse a core-changing edit, but offer no migration help. Protects the contract, but strands the steward at the moment of a legitimate pivot with only a manual path.
- **Guided flow** *(chosen)*: detect a core-changing edit, refuse the in-place rewrite, and actively drive the fork (new bundle + fresh scaffold) and a per-concept migration the steward adjudicates. Costs a real capability in C010, but makes the immutable-core rule self-enforcing and turns the heavyweight pivot into a supported path.

## Decision

Adopt the **guided flow**. C010 - Scaffold distinguishes the immutable core purpose from the revisable periphery, detects when a proposed edit crosses into the core, and — instead of applying it — initiates fork-and-migrate: fork a new bundle with a steward-declared core purpose, then migrate still-in-scope concepts one at a time, with the steward adjudicating each, leaning on O005 portability for the mechanical copy. Periphery refinements still apply in place.

This **supersedes PDR001's consequence** that no capability beyond documentation is required: detection and guidance are the capability that makes the immutable-core contract hold in practice. The product decision itself — core purpose is immutable, a pivot is a new reference — is unchanged, and migration mechanics remain O005 portability with no bespoke persistence.

## Consequences

- Enabling: the immutable-core contract becomes self-enforcing — a core rewrite cannot happen silently, so O008's scope baseline stays stable in practice, not just in principle.
- Enabling: a legitimate pivot is a supported, guided path rather than an undocumented manual chore; still-relevant knowledge migrates via O005 portability with no new persistence machinery.
- Cost: adds detection and guidance behavior to C010, and a core-vs-periphery classification the tool must make; genuinely ambiguous edits are referred to the steward to adjudicate as an act of intent (per PDR001, the line can be fuzzy).
- Cost: revises PDR001's stated "no capability beyond documentation" consequence; this record is the decision trail for that evolution.
