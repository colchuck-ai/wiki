# Requirement → Component map

Every product requirement (from [`docs/product/README.md`](../product/README.md))
maps to exactly one **owner** component, plus any **collaborators** it drives or
reads. No requirement is orphaned; no component is without a requirement (see the
[reverse index](#reverse-index)).

*Owner* = the component accountable for the requirement. *Collaborators* = components
it calls to fulfill it (via the [frozen interfaces](INTERFACES.md)).

## Forward map

| Requirement | Owner | Collaborators |
|---|---|---|
| **O001-R001** Cited claims | C008 Integration | C004 (retain) |
| **O001-R002** Citation resolves to durable form + live locator | C004 Source Retention | C008 (calls at cite time) · ADR003 |
| **O001-R003** Drift detection | C004 Source Retention | C005 (materializes status), C011/C012 (surface to steward) |
| **O002-R001** Recorded recency | C005 Currency & Provenance | C008 (in-file recency stamp) · ADR004 |
| **O002-R002** Change history + decision-provenance | C005 Currency & Provenance | all mutators (report via `record`) |
| **O003-R001** Deprecation over deletion | C008 Integration (`deprecate`) | C011 (decides) |
| **O003-R002** Retirement review | C011 Lifecycle & Retirement | C005 (signals), C002 (escalate), C008 (execute) |
| **O003-R003** Duplication check | C007 Triage | C009 (overlap lookup), C008 (`merge`), C002 (escalate) |
| **O003-R004** Significance bar | C007 Triage | C001 (charter), C006 (`hold_aside`), C002 (escalate) |
| **O004-R001** Intent-driven authoring | C008 Integration | C003 (apply) · ADR001 |
| **O004-R002** Convention encapsulation | C003 OKF Conformance | C008 (calls `apply`) |
| **O004-R003** Assisted upkeep | C011 + C012 (surface) | C008 (carry out by intent), C002 (attention) |
| **O004-R004** Delegated autonomy | C002 Governance | C001 (reads charter) · ADR002 |
| **O005-R001** Navigable index | C009 Discovery & Index | — |
| **O005-R002** Reasoned cross-links | C009 (graph) | C008 (writes links at authoring) |
| **O005-R003** Referential integrity | C009 (`inbound_links`/`dangling`) | C011 (surfaces before retire, repairs via C008) |
| **O005-R004** Question-based retrieval | C010 Question Retrieval | — · ADR005 |
| **O006-R001** Plain-text corpus | C008 Integration (persists) | architecture substrate invariant |
| **O006-R002** Open-format conformance | C003 OKF Conformance (`validate`) | — |
| **O006-R003** Version-control-native | C008 (persists diffable/mergeable) | git substrate invariant |
| **O007-R001** Producer-agnostic intake | C006 Intake & Held-Aside (`submit`) | — |
| **O007-R002** Coverage review | C012 Coverage & Scope Review | C001, C009, C006 (held-aside count), C002 |
| **O007-R003** Recoverable rejection | C006 Intake & Held-Aside (`hold_aside`/`restore`) | C007 (significance decision) |
| **O008-R001** Charter-anchored scope | C001 Charter | — |
| **O008-R002** Scope-aware triage | C007 Triage | C001 (charter), C002 (escalate) |
| **O008-R003** Scope reconciliation | C012 Coverage & Scope Review | C001, C008 (`flag`), C002 (escalate) |
| **O009-R001** Source-grounded authoring | C008 Integration (grounding seam) | C004 (resolve), C002 (admit/hold/escalate) |
| **O009-R002** Legible grounding + review tier | C008 Integration (per-claim stamp) | — · ADR004 |
| **O009-R003** Distillation review | C008 (escalate claim + source span) + C002 (review surface) — *shared, see notes* | — |

## Foundational concepts → owners

| Concept | Owner |
|---|---|
| Charter (required policy anchor) | C001 Charter |
| Delegation envelope (optional policy anchor) | C002 Governance |
| Decision-provenance | C005 Currency & Provenance |
| Review tier | C008 Integration (stamps per-claim) |
| Structured substrate / dual surface | C009 (machine-readable + rendering); machine-readable-first honored by all |

## Reverse index

Every component owns at least one requirement — no shallow/pass-through component:

| Component | Owns |
|---|---|
| C001 Charter | O008-R001 |
| C002 Governance | O004-R004, O009-R003 (attention surface) |
| C003 OKF Conformance | O004-R002, O006-R002 |
| C004 Source Retention | O001-R002, O001-R003 |
| C005 Currency & Provenance | O002-R001, O002-R002 |
| C006 Intake & Held-Aside | O007-R001, O007-R003 |
| C007 Triage | O003-R003, O003-R004, O008-R002 |
| C008 Integration | O001-R001, O003-R001, O004-R001, O005-R002 (write), O006-R001/R003, O009-R001/R002, O009-R003 (escalate half) |
| C009 Discovery & Index | O005-R001, O005-R002, O005-R003 |
| C010 Question Retrieval | O005-R004 |
| C011 Lifecycle & Retirement | O003-R002, O004-R003, O005-R003 (repair) |
| C012 Coverage & Scope Review | O004-R003, O007-R002, O008-R003 |

## Shared-ownership notes (intentional, not orphans)

- **O004-R003** is genuinely shared: C011 and C012 *surface* what needs attention;
  C008 *carries out* the change by intent. The requirement's two halves ("surface" +
  "carry out") land in different owners on purpose — that split is the architecture.
- **O009-R003** is genuinely shared the same way: C008 runs the author-time grounding
  check and *escalates* each unsupported claim with the span of the source it was drawn
  from; C002 owns the *review surface* on which the steward confirms it before
  integration. The "escalate the claim+span" half is C008's; the "present it for review"
  half is C002's — both own their half, so the requirement appears under both.
- **O005-R002** and **O005-R003** name C009 as owner of the *graph/queries* but C008
  as the writer of the underlying links — consistent with the sole-writer rule (ADR001).
- **O006-R001/R003** are substrate invariants enforced at the single write path (C008)
  and validated by C003; they are architecture-level and stated in the
  [README invariants](README.md#engineering-invariants).
