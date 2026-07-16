# Wiki Curation Architecture

Wiki is organized as a **curation pipeline over a shared corpus**, surrounded by standing capabilities that keep the corpus trustworthy as it grows. Raw material enters at one producer-agnostic point (Ingestion Queue), is adjudicated by Triage against the charter's scope and other admission criteria, and — once the steward gives direction — is authored into the corpus by the tool (Integration Authoring). Around the pipeline, standing capabilities keep the corpus discoverable, current, provenanced, on-scope, and high-signal (Index & Navigation, Source Revalidation, Currency Tracking, Lifecycle & Retirement, Coverage Review, Charter). Format knowledge lives in exactly one place — OKF Conformance — as the conventions the tool authors to and a validator that checks the corpus, so no other component encodes the format. The steward drives all of it by **intent**, through a curation Agent Skill: the tool surfaces what needs attention and performs the work; the steward supplies judgment. Components read and write concepts directly following the OKF conventions; they do not share mutable state.

## Principles

- **Intent in, work out.** The steward supplies direction and judgment; the tool performs all authoring, linking, metadata, and upkeep changes. No component requires the steward to hand-edit the corpus.
- **One owner of format knowledge.** Format, naming, and structural conventions live only in OKF Conformance — as the conventions the tool authors to and a validator that checks the corpus — so no other component encodes the format.
- **Producer-agnostic intake.** One entry point makes no assumption about who submitted an item, so people, tools, and agents share one path and one adjudication gate.
- **Cite by durable reference.** A concept cites its source by reference; the reference resolves to something version-controlled alongside the corpus, and a snapshot is captured only as a fallback when the source cannot be referenced durably in place. Provenance travels with the concept.
- **Two kinds of link.** Knowledge cross-links between concepts form the navigable graph and stay within the corpus; provenance references from a concept to its source are a separate class that may point outward.
- **Never dead-end the graph — repair to the successor.** Deleting, moving, or merging away a concept always repairs every inbound link that action would otherwise orphan, pointing each at the departing concept's successor: retargeted to the new path on a `move`, redirected to the replacement or merge target on a `delete`-with-replacement or `merge`, or unlinked to plain text on a `delete` with no successor — so traversal never reaches nothing (see [ADR012](drs/ADR012-repair-to-successor-link-model.md)). Deprecating a concept in place needs no repair, since its file simply stays put.
- **Write verbs carry their own effects.** Each concept write — `create`, `revise`, `move`, `delete`, `deprecate` — records its own recency and change history (C007) and regenerates the affected index (C005) as part of the verb, so no caller orchestrates those steps and they cannot drift out of step with content (see [ADR011](drs/ADR011-concept-verb-surface.md)). Atomicity composes: the `merge` composition (`revise` + `delete`) commits as one unit or not at all — one git commit or none — so it can never half-complete into a duplicate (see [ADR014](drs/ADR014-merge-composition-integrity.md)).
- **The charter is the single scope authority.** Both per-item triage and periodic reconciliation adjudicate against the one declared charter; its core purpose is revisable in place.

## Constraints

- The corpus must remain plain-text and version-control-native at all times; no component may introduce a proprietary or tool-locked store. A captured source snapshot may be a version-controlled binary asset when the source is non-text — a bounded fallback for provenance, not a store.
- Wiki does not define or version the knowledge format. It consumes a published, versioned OKF; a format change is an OKF change, not a Wiki change.
- Integration is never automatic. Material reaches the corpus only after the steward's direction; out-of-scope or un-adjudicated items wait for judgment.
- Wiki targets a **low-volume, single-steward operating envelope**: every item is triaged and every change reviewed by hand, and the steward's judgment capacity is the system's backpressure.

## Technology Choices

- **Git-tracked plain-text files** as the corpus substrate: directly satisfies O006-R003 (version-control-native) — the corpus is diffable and mergeable with no bespoke persistence layer — and holds the plain-text files C004 keeps conformant.
- **The Open Knowledge Format (OKF)** as the storage and structural spec: the format is a product non-goal for Wiki to define, so the architecture depends on OKF rather than inventing one. The version Wiki builds against is pinned in-repo at [`vendor/okf/SPEC.md`](../../vendor/okf/SPEC.md) (OKF 0.1); C004 - OKF Conformance is the only component that reads it.
- **Delivery as a curation Agent Skill** exposing intent-shaped operations: the steward works by intent while the agent performs the authoring, linking, metadata, and upkeep. The operations realize the product's workflows — building and integrating material (Ingest), and keeping the corpus healthy by surfacing what is inaccurate, drifted, or missing (Survey and Lint). Concretely, the skill exposes a small concept **write**-verb surface organized by the steward's intent ([ADR011](drs/ADR011-concept-verb-surface.md)): `create`, `revise`, `move`, `delete`, and `deprecate`, with `merge` as a `revise`+`delete` composition. The source of material — an admitted task, another concept, or the steward's direction — is a parameter of `create`/`revise`, not a separate verb; materialization is a verb effect, not a caller step. Reads are not curation verbs: the Query operation reads the plain-text concepts and the materialized index directly, and the reads that compute (`history`, `inbound_links`, `dangling_links`, `graph`) live on their components — querying the corpus is a consumption convenience, not a curation operation.

## Operations

The curation Agent Skill exposes four steward-facing operations over the components below. Only **Ingest** and the remediation verbs write; **Survey** and **Lint** mutate nothing — they surface findings for the steward, who then directs a write verb to act (recommend-and-confirm; intent in, work out). **Query** is read-only consumption. The write verbs — `create`, `revise`, `move`, `delete`, `deprecate`, and the `merge` composition — live on C003 and C008, and each carries its own recency, history, index, and link-repair effects ([ADR011](drs/ADR011-concept-verb-surface.md)).

- **Ingest** — build and integrate new material. C001 - Ingestion Queue records and durably holds a submission; C002 - Triage recommends a disposition and records the steward's decision; on admission C003 - Integration Authoring `create`s a new concept or `revise`s an existing one (and `merge`s two existing concepts when the steward is consolidating duplicates).
- **Survey** — keep the corpus true, current, on-scope, and complete *over time* (substance and currency, needing the steward's judgment). A read-only aggregation of the components' detect faces: source drift (`C006.revalidate_all`), staleness and recent change (`C007.recency` / `history_all`), retirement candidates from staleness + drift + supersession (`C008.retirement_candidates`), scope drift (`C010.reconcile`), and coverage gaps (C009 - Coverage Review). Each finding routes to a write verb the steward directs — drift → `revise`; retirement candidate or out-of-scope → `delete` / `deprecate` / `move`; gap → `create` — none of which Survey performs itself.
- **Lint** — check that the corpus is well-formed and internally consistent *right now* (form and consistency, mechanical and mostly objective). OKF structural conformance (`C004.validate`), referential integrity (`C005.dangling_links`), and index/recency freshness (`C005.reindex` idempotence, `C007.recency`). A finding is resolved by a `revise` or a mechanical re-render. Because materialization is a verb effect, index and recency can no longer drift from a forgotten call ([ADR011](drs/ADR011-concept-verb-surface.md)) — Lint's freshness check narrows to concepts authored before the tooling or hand-edited outside the verbs.
- **Query** — consume the corpus. Reading a concept or the index is direct file access (the plain-text concepts and the materialized `index.md`); the reads that compute — `C005.graph` / `inbound_links`, `C007.history` — are available for traversal. Query performs no curation write and needs no dedicated verb.

Survey and Lint have no owning component: each is a skill-level aggregation of the detect faces named above, distinct from Ingest, which runs the C001 → C002 → C003 pipeline. The dividing line between them is *substance over time* (Survey) versus *form right now* (Lint).

## Components

### C001 - Ingestion Queue

The single producer-agnostic intake point: records each ingestion task with a reference to the material to distill, ensures that source is durably held — referenced in place when it is version-controlled alongside the corpus, or captured as a snapshot when it is not — and exposes the backlog of un-triaged items.

See [C001 - Ingestion Queue](components/C001-ingestion-queue.md).

### C002 - Triage

Adjudicates each queued item before it enters the corpus, weighing it against several admission criteria — the charter's declared scope, overlap with existing concepts, and whether it is substantial enough to keep — and capturing the steward's disposition, flagging anything that fails a criterion for judgment rather than admitting it automatically.

See [C002 - Triage](components/C002-triage.md).

### C003 - Integration Authoring

Authors concept content through two write verbs — `create` (a new concept from an admitted task) and `revise` (an existing concept in place, from a task, another concept, or the steward's direction) — composing prose, source citations, and reasoned cross-links, and provides the `merge` composition (`revise` + `delete`) that combines two existing concepts.

See [C003 - Integration Authoring](components/C003-integration-authoring.md).

### C004 - OKF Conformance

The single owner of format knowledge: the OKF authoring conventions the tool applies and a validator that checks the corpus against the spec, so no other component and no contributor needs to know the format.

See [C004 - OKF Conformance](components/C004-okf-conformance.md).

### C005 - Index & Navigation

Maintains the navigable index of what the corpus contains, traverses the knowledge cross-link graph, and surfaces the inbound links to a concept before it is deleted, moved, or merged away so C008 can repair them and nothing is left following a dangling reference.

See [C005 - Index & Navigation](components/C005-index-navigation.md).

### C006 - Source Revalidation

Detects when a cited source has changed since a claim was drawn from it — comparing the live source against the durable reference or captured snapshot — and reports drift for the steward rather than repairing it.

See [C006 - Source Revalidation](components/C006-source-revalidation.md).

### C007 - Currency Tracking

Records when each concept last meaningfully changed and maintains a dated change history for each part of the corpus, materialized as in-corpus artifacts so recency and history travel with the bundle.

See [C007 - Currency Tracking](components/C007-currency-tracking.md).

### C008 - Lifecycle & Retirement

Removes or relocates a concept already in the corpus — `delete` removes it outright, `deprecate` marks it in place with a reason and, where one exists, a replacement pointer, and `move` relocates it to a new path — repairing every inbound link the action orphans by the repair-to-successor rule, and surfaces retirement candidates from measurable signals (staleness, source drift, supersession) for the steward's judgment.

See [C008 - Lifecycle & Retirement](components/C008-lifecycle-retirement.md).

### C009 - Coverage Review

Surfaces areas within the intended scope that are thin or absent — from dangling links, the charter's scope, and agent-proposed gaps — as a sourcing agenda for the steward, so gaps do not persist unnoticed.

See [C009 - Coverage Review](components/C009-coverage-review.md).

### C010 - Charter

Holds the steward's declaration of the corpus's purpose and scope as an optional, persistent anchor; applies a core-purpose revision in place; and periodically reconciles the corpus against the declared scope to surface content that has drifted out of it.

See [C010 - Charter](components/C010-charter.md).

## Requirement-Component Map

- **O001-R001 - Cited claims**: C003 - Integration Authoring
- **O001-R002 - Durable source reference**: C001 - Ingestion Queue
- **O001-R003 - Drift detection**: C006 - Source Revalidation
- **O002-R001 - Recorded recency**: C007 - Currency Tracking
- **O002-R002 - Change history**: C007 - Currency Tracking
- **O003-R001 - Deprecation over deletion**: C008 - Lifecycle & Retirement
- **O003-R002 - Retirement review**: C008 - Lifecycle & Retirement
- **O003-R003 - Duplication check**: C002 - Triage
- **O003-R004 - Significance bar**: C002 - Triage
- **O004-R001 - Intent-driven authoring**: C003 - Integration Authoring
- **O004-R002 - Convention encapsulation**: C004 - OKF Conformance
- **O004-R003 - Assisted upkeep**: C006 - Source Revalidation, C007 - Currency Tracking, C008 - Lifecycle & Retirement, C009 - Coverage Review, C010 - Charter
- **O005-R001 - Navigable index**: C005 - Index & Navigation
- **O005-R002 - Reasoned cross-links**: C003 - Integration Authoring, C005 - Index & Navigation
- **O005-R003 - Referential integrity**: C005 - Index & Navigation
- **O006-R001 - Plain-text corpus**: C004 - OKF Conformance
- **O006-R002 - Open-format conformance**: C004 - OKF Conformance
- **O006-R003 - Version-control-native**: satisfied by the git-tracked substrate (see Technology Choices); no dedicated component
- **O007-R001 - Producer-agnostic intake**: C001 - Ingestion Queue
- **O007-R002 - Coverage review**: C009 - Coverage Review
- **O008-R001 - Charter declaration**: C010 - Charter
- **O008-R002 - Scope-aware triage**: C002 - Triage
- **O008-R003 - Scope reconciliation**: C010 - Charter

## See Also

### Architectural Decision Records

- [ADR001 - Single OKF conformance boundary](drs/ADR001-single-okf-conformance-boundary.md)
- [ADR002 - Intake staging and durable-source capture](drs/ADR002-intake-staging-and-durable-source.md)
- [ADR003 - Charter as an in-corpus concept and single scope authority](drs/ADR003-charter-as-in-corpus-concept.md)
- [ADR004 - Triage disposition model](drs/ADR004-triage-disposition-model.md)
- [ADR005 - Integration authoring, provenance, and merge model](drs/ADR005-integration-authoring-provenance-and-merge.md)
- [ADR006 - Index materialization and on-demand graph computation](drs/ADR006-index-materialization-and-graph-computation.md)
- [ADR007 - Citation link form for drift revalidation](drs/ADR007-citation-link-form-for-drift-revalidation.md)
- [ADR008 - Recency materialization and per-directory log scoping](drs/ADR008-recency-materialization-and-log-scoping.md)
- [ADR009 - Retirement, relocation, and link-repair model](drs/ADR009-retirement-relocation-and-link-repair-model.md)
- [ADR010 - Citation pruning on merge reconciliation](drs/ADR010-citation-pruning-on-merge.md)
- [ADR011 - Concept verb surface](drs/ADR011-concept-verb-surface.md)
- [ADR012 - Repair-to-successor link model](drs/ADR012-repair-to-successor-link-model.md)
- [ADR013 - Coverage review signal model and scope-anchored gap suppression](drs/ADR013-coverage-review-signal-model.md)
- [ADR014 - Merge composition integrity](drs/ADR014-merge-composition-integrity.md)

### Change Records

- [CR003 - Concept verb surface: create, revise, merge](../crs/CR003-concept-verb-surface.md)
- [CR005 - Merge composition integrity: content_preserved and atomicity](../crs/CR005-merge-composition-integrity.md)
