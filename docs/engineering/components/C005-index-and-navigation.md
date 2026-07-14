# C005 - Index & Navigation

Maintains the navigable directory listing of what exists and traverses the cross-link graph, and surfaces the inbound links to a concept before it is removed or relocated so no consumer is left following a dangling reference.

Index & Navigation is a standing capability over the corpus. It is the reference's map: what exists (the index) and how concepts relate (the link graph). It reads and regenerates OKF-defined navigation artifacts following C004 - OKF Conformance conventions, and answers inbound-link queries for lifecycle changes. It participates in the **Query** and **Lint** operations (see [ADR010 - Curation Agent Skill and Operations](../drs/ADR010-curation-agent-skill.md)).

## Data model

- **Index** — an OKF §6 `index.md` per directory (including the bundle root): a catalog of the directory's contents, each entry a link plus the linked concept's `description`.
- **Cross-link graph** — the set of OKF §5 markdown links across the corpus, treated as directed edges of an untyped relationship (§5.3). The graph is not a separate store; it is derived by reading the links present in concept bodies.

## Behavior

**Index generation.** On ingest, the affected directories' `index.md` files are regenerated from the concepts' frontmatter `description` fields, following C004 - OKF Conformance conventions (OKF §6). A missing `index.md` may be synthesized on the fly.

**Graph traversal.** Related-concept traversal walks the OKF §5 links as directed edges. Bundle-relative (`/`-prefixed) link targets are preferred as the stable form (§5.1).

**Inbound-link resolution.** To find the links pointing *into* a concept — needed before that concept is relocated or removed — scan the corpus for markdown links whose target resolves to the concept's path. These inbound links are surfaced to C008 - Lifecycle & Retirement so a deprecation or relocation never leaves a dangling reference.

An optional on-device markdown search engine is a scale affordance beyond the index for larger corpora; it is not required for navigation at moderate scale.

## Edge cases

- **Broken / dangling link**: not an error (OKF §5/§9 — it may be not-yet-written knowledge). It is reported to C009 - Coverage Review as a coverage signal, and inbound links are resolved before any relocation or removal so traversal never dead-ends.
- **Missing `index.md`**: synthesized on the fly rather than treated as a fault (OKF §6).

## Relationships

- **C004 - OKF Conformance**: index and link conventions (OKF §5/§6) come from the conformance conventions; index regeneration follows them.
- **C008 - Lifecycle & Retirement**: provides inbound-link resolution before a concept is deprecated or relocated.
- **C009 - Coverage Review**: supplies the represented-concept directory and the dangling-link set as coverage signals.
- **C002 - Scope Triage**: serves index and graph reads used to detect an incoming item's overlap with existing concepts.
- **C010 - Scaffold**: traverses the graph during scope reconciliation and resolves inbound links during a migration.

## Success criteria

- A current `index.md` exists for navigable directories, so a consumer can see what exists before opening documents (O003-R001).
- The cross-link graph is traversable, so consumers can follow relationships rather than only search (O003-R002).
- Before any removal or relocation, the inbound links to the affected concept are surfaced, so no consumer is left following a dangling reference (O003-R004).

## See Also

### Architectural Decision Records

- [ADR010 - Curation Agent Skill and Operations](../drs/ADR010-curation-agent-skill.md)

### Change Records

- [CR008 - Delivery Form and Standing-Capability Specifications](../../crs/CR008-delivery-form-and-standing-capabilities.md)
