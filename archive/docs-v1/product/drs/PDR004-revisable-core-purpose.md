# PDR004 - Revisable Core Purpose

A wiki's core purpose **may be revised in place**. A revision is a deliberate act that triggers a **Survey-driven reconciliation** of the corpus against the new purpose — surface the drift, plan it, execute it — rather than a fork to a new reference. Version control preserves the prior whole-bundle state. This **supersedes [PDR001 - Immutable Core Telos](PDR001-immutable-core-telos.md)**.

## Context

PDR001 fixed the scaffold's core purpose for the life of the reference to guarantee a *durable* scope contract ("if it isn't here, it's out of scope, not missing"), and modeled a genuine pivot as a new bundle plus a migration — realized by [ADR005 - Guided Fork-and-Migrate](../../engineering/drs/ADR005-guided-fork-and-migrate.md).

Working through the fork mechanics exposed two problems. Migrating concepts one at a time across two live bundles **partitions a connected graph**, and a relationship spanning the two halves cannot be a bundle-relative OKF link (SPEC §5.1) — preserving it would require cross-bundle relationships the format does not define. And cloning a whole bundle purely to preserve the prior state **duplicates what version control already provides**. Meanwhile the immutable-core friction is heavier than the value it protects: references legitimately evolve their purpose, and the tool already has Survey to re-adjudicate the corpus against the declared scope. The framework must decide whether the core purpose is truly immutable or revisable with a disciplined reconciliation.

## Options

- **Immutable core, fork-and-migrate** (PDR001/ADR005, now rejected): a durable contract, but forces a heavyweight fork and needs cross-bundle relationships to preserve links that straddle the split.
- **Fork as full copy, then prune**: clone the whole bundle, re-anchor the scaffold, and prune out-of-scope drift. Avoids cross-bundle links, but the clone duplicates state version control already preserves and leaves an abandoned duplicate behind.
- **Revisable core in place, Survey-driven reconciliation** *(chosen)*: edit the core purpose in place; recover the prior state from version control if needed; run Survey to surface content that has drifted from the new purpose (and areas now expected), then the steward plans and executes the reconciliation with normal operations.

## Decision

Adopt the **revisable core in place**. The workflow is: (1) the steward revises the core purpose on the scaffold; (2) the tool runs Survey against the new purpose; (3) the steward plans the reconciliation from Survey's drift-and-gap output; (4) the steward executes it with normal operations — deprecate or retire drifted concepts (C008 - Lifecycle & Retirement), source new gaps through Ingest. The prior whole-bundle state is preserved by version control at the origin. This supersedes PDR001 and is realized by [ADR011 - In-Place Core-Purpose Change](../../engineering/drs/ADR011-in-place-core-purpose-change.md); [ADR005 - Guided Fork-and-Migrate](../../engineering/drs/ADR005-guided-fork-and-migrate.md) is superseded.

## Consequences

- Enabling: a purpose change becomes a normal, in-place cleaning operation rather than a heavyweight fork, and no cross-bundle relationships ever arise.
- Enabling: Survey re-adjudicates the corpus against the new purpose, so past triage (O008-R002) and reconciliation (O008-R003) judgments are explicitly reconciled rather than silently invalidated.
- Cost: the scope contract changes from *durable* to *current-as-of-now* — a consumer relying on "absence means out-of-scope" gets that guarantee as of the current purpose, not for all time. This is exactly the guarantee PDR001 protected; it is traded for honesty about how references evolve and for a far lighter pivot.
- Cost: recoverability of the prior whole-bundle state relies on version control at the origin (OKF recommends git); this is distinct from currency metadata, which remains in-corpus per [ADR006 - In-Corpus Currency](../../engineering/drs/ADR006-in-corpus-currency.md). A consumer holding only a tarball keeps whatever snapshot they took.
- Cost: C010 still needs a core-vs-periphery classification — not to block an edit, but to decide whether a revision warrants the reconciliation workflow; ambiguous cases are referred to the steward (per ADR011).
