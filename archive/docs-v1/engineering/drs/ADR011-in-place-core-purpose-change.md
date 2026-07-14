# ADR011 - In-Place Core-Purpose Change with Survey-Driven Reconciliation

Realize [PDR004 - Revisable Core Purpose](../../product/drs/PDR004-revisable-core-purpose.md) on the engineering side: C010 - Scaffold applies a core-affecting scaffold revision **in place** and initiates a **Survey-driven reconciliation** (revise → Survey → plan → execute), rather than detecting the edit, refusing it, and driving a fork-and-migrate. This **supersedes [ADR005 - Guided Fork-and-Migrate](ADR005-guided-fork-and-migrate.md)**.

## Context

ADR005 made the immutable-core rule self-enforcing: C010 detected a core-changing scaffold edit, refused to apply it in place, and drove a fork (new bundle + fresh scaffold) plus a per-concept migration. PDR004 reverses the immutability decision — the core purpose is now revisable in place, and version control preserves the prior state.

Two facts make fork-and-migrate the wrong mechanism now. Partitioning a live graph across two bundles would need cross-bundle relationships OKF does not define (bundle-relative links, SPEC §5.1, cannot span bundles). And the reconciliation a pivot needs — surface what no longer fits the new purpose, and what is now expected but missing — is exactly what Survey already provides: C010 scope reconciliation plus C009 - Coverage Review, both against the declared scope. The architecture must decide how a core-purpose revision is carried out without a fork.

## Options

- **Detect-and-block + fork-and-migrate** (ADR005, rejected): self-enforcing immutability, but obsolete under PDR004 and dependent on cross-bundle relationships.
- **Apply silently, no reconciliation**: cheapest, but a core change silently leaves drifted content in place with no prompt, so the O008 on-scope baseline degrades unnoticed — the failure O008-RSK001 names.
- **Apply in place + initiate Survey-driven reconciliation** *(chosen)*: detect that a revision affects the core, apply it, and immediately run Survey against the new purpose so the induced drift and new gaps surface for the steward to plan and execute.

## Decision

Adopt **in-place revision with Survey-driven reconciliation**. C010 distinguishes a core-affecting revision from a periphery refinement. A periphery refinement applies in place unchanged. A core-affecting revision applies in place and initiates the reconciliation workflow: run Survey (scope reconciliation surfaces content now out of scope; coverage review surfaces areas now expected but thin or absent), the steward plans, then executes through normal operations — deprecate or retire drifted concepts via C008 - Lifecycle & Retirement, source gaps through Ingest. No fork, no new bundle, no cross-bundle relationships; the prior whole-bundle state is recoverable via version control. Supersedes ADR005.

## Consequences

- Enabling: a purpose pivot reuses Survey and existing operations — no fork machinery and no cross-bundle link semantics to define.
- Enabling: the drift a revision induces is surfaced and reconciled explicitly, keeping O008's on-scope metric honest after a pivot rather than silently stale.
- Cost: C010 still makes a core-vs-periphery classification (now to decide whether to trigger reconciliation, not to block), a fuzzy judgment referred to the steward when ambiguous — consistent with PDR004 and the prior ADR005/PDR001 acknowledgment that the line can be fuzzy.
- Cost: a large pivot can surface a large reconciliation backlog in one Survey; within the low-volume single-steward envelope ([ADR009](ADR009-low-volume-operating-envelope.md)) the steward works through it by hand.
- Cost: because the graph is never partitioned, inbound-link resolution across bundles is no longer needed; ordinary deprecation-with-pointer (C008, O003-R004) handles any concept retired during reconciliation, so "never dead-end the graph" is upheld without fork-time link surgery.
