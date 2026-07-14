# CR005 - Conformance Boundary Re-scope

Re-scoped **C004** from a mandatory runtime read/write gateway to the single owner of OKF format knowledge, realized as authoring conventions (applied by the curation Agent Skill) plus a conformance validator. Promoted C004 from Tier 0 to Tier 1, renamed it **C004 - OKF Conformance** (dropping "Adapter"), reframed the architecture's "One format boundary" principle as "one owner of format knowledge," and superseded [ADR001 - Single OKF Conformance Boundary](../engineering/drs/ADR001-single-okf-conformance-boundary.md). Scoped to the engineering architecture document, C004, and the components whose corpus access was described as passing "through the Adapter."

## Change

Before, [ADR001](../engineering/drs/ADR001-single-okf-conformance-boundary.md) mandated that every corpus read and write route through C004 as a runtime gateway, and every other component "reasons about concepts, never files." The C004 block, the "One format boundary" principle, and the relationships of C001, C003, C005, C007, and C008 all encoded that mandatory indirection.

After, C004 owns the OKF authoring conventions the Agent Skill applies and provides a conformance validator that checks the bundle against OKF §9. Components author and read the corpus directly following those conventions; conformance is enforced by validation at defined checkpoints (after ingest, during `lint`) rather than by a mandatory data path. The "One format boundary" principle is reframed as **one owner of format knowledge**, and C004 is promoted to its own Tier-1 document.

## Rationale

OKF §9 conformance is near-empty and permissive — parseable frontmatter with a non-empty `type`, plus §6/§7 structure for `index.md`/`log.md` when present — and the format is designed to stay stable. A mandatory runtime gateway is an over-abstraction whose per-access indirection cost the format's thinness does not earn, especially when the authoring engine is an LLM that already encapsulates the conventions the gateway would apply. Concentrating format knowledge in one place is still worth keeping; a runtime chokepoint to achieve it is not.

## Affects

- Engineering: the architecture document (the "One format boundary" principle, the C004 Technology Choice line, the C004 component block, and the relationships that said "through C004"/"through the Adapter"), C004 - OKF Conformance (new Tier-1 component document, renamed), and the wording of C001 - Inbox and C003 - Integration Authoring ("through the Adapter" → "following OKF conventions owned by C004").
- Decision trail: supersedes [ADR001 - Single OKF Conformance Boundary](../engineering/drs/ADR001-single-okf-conformance-boundary.md); captured by [ADR007 - Conformance as Conventions and Validation](../engineering/drs/ADR007-conformance-conventions-and-validation.md).
