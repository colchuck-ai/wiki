# CR008 - Delivery Form and Standing-Capability Specifications

Defined Wiki's delivery form and operation surface, pinned the meaning of
"bundle," and promoted the four standing-capability components
(C002 - Scope Triage, C005 - Index & Navigation, C006 - Source Revalidation,
C009 - Coverage Review) from inline Tier-0 statements to their own documents with
their approaches specified. Scoped to the engineering architecture document and
those four components, plus C010 - Scaffold for the bundle definition.

## Change

- **Delivery form**: the architecture's Technology Choices previously committed
  only to "an Agent Skill" with no operation surface. Added the curation Agent
  Skill's four operations — **Ingest**, **Query**, **Survey**, **Lint** — as the
  steward's interaction surface (captured by [ADR010 - Curation Agent Skill and Operations](../engineering/drs/ADR010-curation-agent-skill.md)).
  Scaffold-relative assessment — coverage-gap review (C009 - Coverage Review) and
  scope reconciliation (C010 - Scaffold) — runs under **Survey**, whose coverage
  output is a sourcing agenda; the corrective hygiene checks (conformance,
  referential integrity, revalidation, staleness, retirement) run under **Lint**.
- **"Bundle" definition**: previously undefined in-repo; now pinned to the
  **OKF Knowledge Bundle** (SPEC §2/§3). C010 - Scaffold and ADR005 fork-and-migrate
  now reference a concretely defined unit.
- **C002 - Scope Triage**: promoted Tier 0 → Tier 1. Overlap detection specified
  as index-scan (OKF §6 `index.md`) to shortlist candidates, with an optional
  on-device markdown search engine as a scale affordance; scope adjudication is a
  scaffold-anchored Agent-Skill judgment.
- **C005 - Index & Navigation**: promoted Tier 0 → Tier 1. Index generation from
  concept frontmatter (`description`) per OKF §6; cross-link graph as OKF §5 links
  treated as directed edges; inbound-link resolution by scanning link targets.
- **C006 - Source Revalidation**: promoted Tier 0 → Tier 1. Re-fetch the cited
  source, normalize it as at capture, and diff against the captured-source
  snapshot (C001); a non-empty diff is drift.
- **C009 - Coverage Review**: promoted Tier 0 → Tier 1. "Expected coverage" input
  assembled from three signals — dangling links (OKF §5/§9 not-yet-written
  knowledge), the scaffold's declared scope, and agent-proposed gaps (optionally
  via web search).
- **Architecture document**: Technology Choices gains the operation surface; the
  C002/C005/C006/C009 blocks collapse to reference form (one-liner plus a See
  link) as those components move to their own documents.

Before, the delivery form was a bare "Agent Skill," "bundle" was undefined, and
these four components were one-line statements with unspecified approaches. After,
the steward has a defined Ingest/Query/Lint surface, "bundle" resolves to the
vendored OKF Knowledge Bundle, and each component states a concrete,
buildable approach.

## Rationale

These are the operational-detail items the implementation-gaps assessment
deliberately deferred at the intent level. Vendoring OKF (which defines the
bundle, the index, the log, and the permissive link semantics) plus a single
delivery-form decision makes them resolvable without inventing anything: each
component's approach is an engineering realization of a one-liner that already
traced cleanly to its outcome. Promoting the four components gives their approach,
edge cases, and success criteria a home without changing any requirement.

## Affects

- Engineering: the architecture document (Technology Choices, and the
  C002/C005/C006/C009 component blocks become references); C002, C005, C006, C009
  (new component documents); C010 - Scaffold (bundle definition reference).
- No product requirement or outcome text changes — the operations and approaches
  realize requirements that already exist (O002-R003, O003-R001/R002/R004,
  O001-R002, O008-R002).
- Captured by [ADR010 - Curation Agent Skill and Operations](../engineering/drs/ADR010-curation-agent-skill.md).
