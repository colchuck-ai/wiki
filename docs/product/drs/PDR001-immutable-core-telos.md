# PDR001 - Immutable Core Telos

A wiki's **core purpose is fixed for the life of the reference**. The scaffold that anchors on-scope focus (O008) may have its peripheral boundaries and context refined over time, but its core statement of purpose is not rewritten in place. A genuine change of purpose constitutes a *new* reference — into which still-relevant knowledge is migrated — not an edit to the existing one.

## Context

O008 (On-scope focus) introduces an optional **scaffold**: a persistent anchor, articulated by the steward and maintained by the tool, that keeps an otherwise emergent knowledge graph tethered to its intended purpose. O008-R001 requires that anchor to endure for the life of the reference.

Persistence forces a question the framework must answer explicitly: may the scaffold's core purpose be rewritten later? The scaffold's entire value is as a *stable* baseline. O008 measures the proportion of the reference that has drifted outside its intended purpose; every triage decision (O008-R002) and reconciliation (O008-R003) is a judgment made against that purpose. If the core purpose were a moving target, those judgments would be retroactively meaningless and consumers could no longer trust the scope contract ("if it isn't here, it's out of scope, not missing").

## Options

- **Freely revisable scaffold**: the steward may rewrite purpose and boundaries at any time. Simplest, but the anchor drifts — O008's fidelity metric has no stable baseline, and past curation decisions lose their meaning.
- **Immutable core, revisable periphery** *(chosen)*: the core statement of purpose is fixed once declared; only boundary detail and supporting context evolve. A core-purpose change is modeled as a new reference plus a migration.
- **Versioned telos**: retain a dated history of purpose revisions. More machinery than the value warrants; consumers would have to reason about which telos applied when.

## Decision

Adopt **immutable core, revisable periphery**. The scaffold's core purpose is set at declaration and honored for the reference's life; peripheral boundaries and context may be refined as understanding deepens. A true pivot in purpose is not an in-place edit — it is a new reference, and still-relevant concepts are migrated across using the portability the format already guarantees (O005: plain-text, OKF-conformant, version-control-native concepts copy cleanly between bundles). This keeps O008's baseline stable and preserves the scaffold as both a durable anchor and a trustworthy scope contract.

## Consequences

- Enabling: O008's "proportion outside intended purpose" has a stable baseline, so triage (O008-R002) and reconciliation (O008-R003) judgments stay valid over time.
- Enabling: consumers can rely on the scope contract durably — absence means out-of-scope, not merely unwritten.
- Enabling: migration needs no bespoke machinery; it falls out of O005 portability. No new product capability is required beyond documenting the workflow (fork the bundle, carry over in-scope concepts, re-anchor the scaffold).
- Cost: a genuine change of purpose is comparatively heavyweight — a migration rather than an edit. This is intentional friction that protects the anchor's meaning.
- Cost: the scaffold must distinguish "core purpose" (fixed) from "peripheral boundary/context" (revisable), a line that can be fuzzy; the steward adjudicates it as an act of intent.
