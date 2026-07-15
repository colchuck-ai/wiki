# Wiki Curation Architecture

Wiki is organized as a **curation pipeline over a shared corpus**, surrounded by standing capabilities that keep the corpus trustworthy as it grows. Raw material enters at one producer-agnostic point (Ingestion Queue), is adjudicated by Triage against the charter's scope and other admission criteria, and — once the steward gives direction — is authored into the corpus by the tool (Integration Authoring). Around the pipeline, standing capabilities keep the corpus discoverable, current, provenanced, on-scope, and high-signal (Index & Navigation, Source Revalidation, Currency Tracking, Lifecycle & Retirement, Coverage Review, Charter). Format knowledge lives in exactly one place — OKF Conformance — as the conventions the tool authors to and a validator that checks the corpus, so no other component encodes the format. The steward drives all of it by **intent**, through a curation Agent Skill: the tool surfaces what needs attention and performs the work; the steward supplies judgment. Components read and write concepts directly following the OKF conventions; they do not share mutable state.

## Principles

- **Intent in, work out.** The steward supplies direction and judgment; the tool performs all authoring, linking, metadata, and upkeep changes. No component requires the steward to hand-edit the corpus.
- **One owner of format knowledge.** Format, naming, and structural conventions live only in OKF Conformance — as the conventions the tool authors to and a validator that checks the corpus — so no other component encodes the format.
- **Producer-agnostic intake.** One entry point makes no assumption about who submitted an item, so people, tools, and agents share one path and one adjudication gate.
- **Cite by durable reference.** A concept cites its source by reference; the reference resolves to something version-controlled alongside the corpus, and a snapshot is captured only as a fallback when the source cannot be referenced durably in place. Provenance travels with the concept.
- **Two kinds of link.** Knowledge cross-links between concepts form the navigable graph and stay within the corpus; provenance references from a concept to its source are a separate class that may point outward.
- **Never dead-end the graph.** Removal is modeled as deprecation with a replacement pointer, and inbound links are resolved before any removal or relocation, so traversal never reaches nothing.
- **The charter is the single scope authority.** Both per-item triage and periodic reconciliation adjudicate against the one declared charter; its core purpose is revisable in place.

## Constraints

- The corpus must remain plain-text and version-control-native at all times; no component may introduce a proprietary or tool-locked store. A captured source snapshot may be a version-controlled binary asset when the source is non-text — a bounded fallback for provenance, not a store.
- Wiki does not define or version the knowledge format. It consumes a published, versioned OKF; a format change is an OKF change, not a Wiki change.
- Integration is never automatic. Material reaches the corpus only after the steward's direction; out-of-scope or un-adjudicated items wait for judgment.
- Wiki targets a **low-volume, single-steward operating envelope**: every item is triaged and every change reviewed by hand, and the steward's judgment capacity is the system's backpressure.

## Technology Choices

- **Git-tracked plain-text files** as the corpus substrate: directly satisfies O006-R003 (version-control-native) — the corpus is diffable and mergeable with no bespoke persistence layer — and holds the plain-text files C004 keeps conformant.
- **The Open Knowledge Format (OKF)** as the storage and structural spec: the format is a product non-goal for Wiki to define, so the architecture depends on OKF rather than inventing one. The version Wiki builds against is pinned in-repo at [`vendor/okf/SPEC.md`](../../vendor/okf/SPEC.md) (OKF 0.1); C004 - OKF Conformance is the only component that reads it.
- **Delivery as a curation Agent Skill** exposing intent-shaped operations: the steward works by intent while the agent performs the authoring, linking, metadata, and upkeep. The operations realize the product's workflows — building and integrating material (Ingest), and keeping the corpus healthy by surfacing what is inaccurate, drifted, or missing (Survey and Lint). Querying the corpus is a consumption convenience, not a curation operation.

## Components

### C001 - Ingestion Queue

The single producer-agnostic intake point: records each ingestion task with a reference to the material to distill, ensures that source is durably held — referenced in place when it is version-controlled alongside the corpus, or captured as a snapshot when it is not — and exposes the backlog of un-triaged items.

See [C001 - Ingestion Queue](components/C001-ingestion-queue.md).

### C002 - Triage

Adjudicates each queued item before it enters the corpus, weighing it against several admission criteria — the charter's declared scope, overlap with existing concepts, and whether it is substantial enough to keep — and capturing the steward's disposition, flagging anything that fails a criterion for judgment rather than admitting it automatically. When an item overlaps existing concepts, it surfaces them so a merge can be steered against a concrete target.

### C003 - Integration Authoring

Turns a triaged item and the steward's direction into a concept — composing it, attaching its source citation, and cross-linking related concepts with the reason for each stated — and executes the steward-chosen reconciliation when the item merges with an existing concept.

### C004 - OKF Conformance

The single owner of format knowledge: the OKF authoring conventions the tool applies and a validator that checks the corpus against the spec, so no other component and no contributor needs to know the format.

See [C004 - OKF Conformance](components/C004-okf-conformance.md).

### C005 - Index & Navigation

Maintains the navigable index of what the corpus contains, traverses the knowledge cross-link graph, and surfaces the inbound links to a concept before it is removed or relocated so nothing is left following a dangling reference.

### C006 - Source Revalidation

Detects when a cited source has changed since a claim was drawn from it — comparing the live source against the durable reference or captured snapshot — and reports drift for the steward rather than repairing it.

### C007 - Currency Tracking

Records when each concept last meaningfully changed and maintains a dated change history for each part of the corpus, materialized as in-corpus artifacts so recency and history travel with the bundle.

### C008 - Lifecycle & Retirement

Marks a concept deprecated or superseded — with a reason and, where one exists, a replacement pointer — rather than deleting it, and surfaces retirement candidates from measurable signals (staleness, source drift, supersession) for the steward's judgment.

### C009 - Coverage Review

Surfaces areas within the intended scope that are thin or absent — from dangling links, the charter's scope, and agent-proposed gaps — as a sourcing agenda for the steward, so gaps do not persist unnoticed.

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
