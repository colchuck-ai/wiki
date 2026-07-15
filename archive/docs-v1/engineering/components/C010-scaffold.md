# C010 - Scaffold

Lets the steward declare the reference's purpose and scope as an optional, persistent anchor that endures for the life of the reference; distinguishes the core purpose from the periphery; applies a core-purpose revision in place and initiates a Survey-driven reconciliation of the corpus against the new purpose; and supports periodically reconciling the graph against the declared scope to surface content that has drifted out of scope.

## Data model

The scaffold is a plain-text, version-controlled artifact that travels with the bundle like the rest of the corpus (O005) — realized as a first-class OKF concept (for example `scaffold.md` with `type: Scaffold`) at the bundle root, so it is linkable and reconcilable like any other concept and needs no store outside the corpus. A **bundle** is an OKF Knowledge Bundle (SPEC §2/§3): a directory tree of markdown distributed as a git repository, tarball, or subdirectory (see [ADR010 - Curation Agent Skill and Operations](../drs/ADR010-curation-agent-skill.md)). The scaffold holds two parts with different mutability:

- **Core purpose** — the statement of what the reference is for. Revisable in place; a revision triggers a Survey-driven reconciliation of the corpus against the new purpose (see [PDR004 - Revisable Core Purpose](../../product/drs/PDR004-revisable-core-purpose.md)).
- **Periphery** — supporting boundaries and context. Revisable as understanding deepens, with no reconciliation needed.

The scaffold is optional. With none declared, there is no scope contract for triage to adjudicate against and no core-affecting-change detection applies.

## Behavior

**Declaration.** The steward articulates the purpose — from a single statement of intent to a fuller set of boundaries and context — and the tool maintains it thereafter. The steward never hand-edits the corpus to establish scope; they declare intent and the tool holds the anchor.

**Periphery refinement.** Edits that refine boundary detail or context are applied in place, with no reconciliation needed. The core purpose is untouched.

**Core-purpose revision and Survey-driven reconciliation.** When a proposed edit alters the core statement of purpose rather than the periphery, the tool applies it in place and initiates the reconciliation workflow (see [ADR011 - In-Place Core-Purpose Change](../drs/ADR011-in-place-core-purpose-change.md)):

1. Apply the revised core purpose to the scaffold in place. The prior whole-bundle state remains recoverable from version control.
2. Run **Survey** against the new purpose: scope reconciliation surfaces content now out of scope, and coverage review (C009 - Coverage Review) surfaces areas now expected but thin or absent.
3. The steward plans the reconciliation from Survey's output and executes it with normal operations — deprecate or retire drifted concepts via C008 - Lifecycle & Retirement, and source new gaps through Ingest.

No fork, no new bundle, and no cross-bundle relationships: the graph is never partitioned, so it needs no cross-bundle link semantics.

**Scope reconciliation.** Periodically traverse the graph (via C005 - Index & Navigation) against the declared scope to surface content that has drifted out of scope, presenting it for the steward's judgment rather than acting automatically. This runs as part of the **Survey** operation — the scaffold-relative assessment — paired with coverage-gap review (C009 - Coverage Review): the two directions of measuring the corpus against its declared purpose (drifted-out vs. missing), distinct from the corrective Lint hygiene pass (see [ADR010 - Curation Agent Skill and Operations](../drs/ADR010-curation-agent-skill.md)).

## Edge cases

- **Core-vs-periphery ambiguity**: when the tool cannot tell whether an edit refines the periphery or revises the core, it flags the edit as potentially core-affecting and asks the steward to adjudicate whether the reconciliation workflow should run. Per PDR004 the line can be fuzzy and is settled as an act of steward intent.
- **Large reconciliation backlog after a revision**: a broad core-purpose revision can surface a large drift-and-gap set in one Survey; within the low-volume single-steward envelope ([ADR009 - Low-Volume Operating Envelope](../drs/ADR009-low-volume-operating-envelope.md)) the steward works through it by hand rather than the tool acting automatically.
- **Concept drifted out of scope but with inbound links**: it is retired as an ordinary deprecation-with-pointer (C008), with inbound links resolved first (via C005), so reconciliation never leaves a dangling reference.

## Relationships

- **C002 - Scope Triage**: provides the scope contract each incoming item is adjudicated against.
- **C005 - Index & Navigation**: traverses the graph during reconciliation to find drifted content, and resolves inbound links before a drifted concept is retired.
- **C009 - Coverage Review**: paired with scope reconciliation under Survey — the missing-coverage direction of measuring the corpus against the declared purpose, and the surfacing of gaps a core-purpose revision creates.
- **C008 - Lifecycle & Retirement**: retires or deprecates concepts the steward removes during a reconciliation.

## Success criteria

- A core-purpose revision is detected and, when applied, always initiates the Survey-driven reconciliation — the corpus is never left silently drifted against a changed purpose (O008-R001, O008-R003, PDR004).
- Peripheral boundary and context can be refined in place with no reconciliation.
- Reconciliation retires drifted concepts as deprecations-with-pointers, leaving no dangling reference and requiring no cross-bundle relationships.
- Scope reconciliation surfaces out-of-scope drift against the declared scope (O008-R003) rather than leaving it to persist unnoticed.

## See Also

### Architectural Decision Records

- [ADR011 - In-Place Core-Purpose Change](../drs/ADR011-in-place-core-purpose-change.md)
- [ADR005 - Guided Fork-and-Migrate](../drs/ADR005-guided-fork-and-migrate.md) (superseded by ADR011)
- [ADR010 - Curation Agent Skill and Operations](../drs/ADR010-curation-agent-skill.md)

### Change Records

- [CR002 - Steward-Attention Interactions](../../crs/CR002-steward-attention-interactions.md)
- [CR008 - Delivery Form and Standing-Capability Specifications](../../crs/CR008-delivery-form-and-standing-capabilities.md)
- [CR011 - Revisable Core Purpose](../../crs/CR011-revisable-core-purpose.md)
