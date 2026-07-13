# Wiki Curation Architecture

Wiki is organized as a **curation pipeline over a shared corpus**, with a single format boundary. Raw material enters at one producer-agnostic point (the Inbox), is adjudicated against the reference's declared scope (Scope Triage), and — once the steward supplies intent — is authored into the corpus by the tool (Integration Authoring). Every read and write of the corpus passes through the **OKF Conformance Adapter**, so no other component encodes format knowledge: the on-disk format is deferred entirely to the Open Knowledge Format, exactly as the product intends. Around this pipeline sit standing capabilities that keep the corpus discoverable, current, provenanced, on-scope, and high-signal (Index & Navigation, Source Revalidation, Currency Tracking, Lifecycle & Retirement, Coverage Review, Scaffold). Components communicate by reading and writing concepts through the Adapter and by passing items along the pipeline; they do not share mutable state directly.

## Principles

- **One format boundary.** All corpus reads and writes go through the OKF Conformance Adapter. Format, naming, and structural conventions live in exactly one place, so the rest of the system reasons about concepts, never about files.
- **Intent in, authoring out.** Steward-facing components capture judgment and direction (keep, merge, deprecate, discard); tool-facing components perform all authoring, linking, and metadata. No component requires the steward to hand-edit the corpus.
- **Producer-agnostic intake.** The Inbox is the sole entry point and makes no assumption about who deposited an item, so automated producers and humans share one path and one adjudication gate.
- **Provenance and recency travel with the concept.** A statement's originating source and its last-meaningful-change time are first-class metadata on the concept, not external bookkeeping, so a consumer can check both at the moment of reliance.
- **Never dead-end the graph.** Removal is modeled as deprecation with a replacement pointer, and any relocation or removal first resolves inbound links, so traversal never reaches nothing.
- **The scaffold is the single scope authority.** Both per-item triage and periodic reconciliation adjudicate against the one declared scaffold, whose core purpose is fixed for the life of the reference.

## Constraints

- The corpus must remain plain-text and version-control-native at all times; no component may introduce a binary or tool-proprietary store.
- Wiki does not define or version the knowledge format. It consumes a published, versioned OKF; a format change is an OKF change, not a Wiki change.
- The scaffold's core purpose is immutable for the life of the reference (see [PDR001 - Immutable Core Telos](../product/drs/PDR001-immutable-core-telos.md)). A genuine change of purpose is a new bundle plus a migration, not an in-place edit, so no component offers a "rewrite purpose" operation.
- Integration is never automatic. An item reaches the corpus only after the steward's keep or merge direction; out-of-scope or unadjudicated items wait for judgment.

## Technology Choices

- Git-tracked plain-text files as the corpus substrate: satisfies plain-text and version-control-native storage directly and needs no bespoke persistence layer.
- The Open Knowledge Format (OKF) as the storage and structural spec: the format is a product non-goal for Wiki to define, so the architecture depends on OKF rather than inventing a format.
- Agent-driven authoring (realized as an Agent Skill): the steward works by intent while an agent performs authoring, linking, and metadata, keeping hands-on curation effort flat as material scales.

## Components

### C001 - Inbox

The single, producer-agnostic capture point where raw material is recorded — with its origin — the moment it arrives, retained as durable provenance after processing, and where the count and age of un-integrated items are exposed.

See [C001 - Inbox](components/C001-inbox.md).

### C002 - Scope Triage

Adjudicates each inbox item against the declared scaffold scope, capturing the steward's keep, merge, deprecate, or discard direction and flagging out-of-scope items for judgment rather than integrating them automatically. When an item overlaps an existing concept, surfaces the overlapping concept(s) so a merge direction can be steered against a concrete target rather than issued blind.

**Relationships**

- **C010 - Scaffold**: reads the declared purpose and scope to evaluate each item.
- **C003 - Integration Authoring**: forwards kept or merged items, with steward intent and the set of overlapping concepts a merge must reconcile against, for authoring.

### C003 - Integration Authoring

Turns a triaged item plus steward intent into corpus content — composing the concept, attaching its source link, and cross-linking related concepts with each reason stated — and, when an item overlaps an existing concept, executes the steward-chosen reconciliation rather than authoring blindly.

See [C003 - Integration Authoring](components/C003-integration-authoring.md).

### C004 - OKF Conformance Adapter

The single gateway through which every corpus read and write passes. Enforces the versioned open knowledge format — plain-text representation, structural and naming/identifier conventions, and open-format conformance — and keeps the corpus diffable and mergeable in standard version control, so no other component and no contributor need know the format's conventions.

**Relationships**

- **C005 - Index & Navigation**: serves concept and link data for indexing and traversal.

**Architectural Decision Records**

- [ADR001 - Single OKF Conformance Boundary](drs/ADR001-single-okf-conformance-boundary.md)

### C005 - Index & Navigation

Maintains the navigable directory listing of what exists and traverses the cross-link graph, and surfaces the inbound links to a concept before it is removed or relocated so no consumer is left following a dangling reference.

**Relationships**

- **C008 - Lifecycle & Retirement**: provides inbound-link resolution before a concept is deprecated or relocated.

### C006 - Source Revalidation

Detects when a cited source has changed since a statement was captured, so a curator can tell which statements no longer match the source they cite and which underlying sources have moved on.

**Relationships**

- **C001 - Inbox**: reads the captured-source snapshot as the baseline it diffs the live source against.
- **C007 - Currency Tracking**: a detected drift is the signal that a concept is due for reconciliation.

### C007 - Currency Tracking

Records when each concept last meaningfully changed and maintains a dated, chronological history of updates for each scope of the reference, so recency and change history are legible to any consumer.

**Relationships**

- **C003 - Integration Authoring**: records the change event when a concept is authored or updated.

### C008 - Lifecycle & Retirement

Marks a concept as deprecated or superseded — with a reason and, where one exists, a pointer to its replacement — rather than hard-deleting it, and supports periodically identifying concepts no longer relied upon as candidates for pruning.

**Relationships**

- **C003 - Integration Authoring**: deprecates a superseded statement, with a replacement pointer, when a merge reconciliation supersedes it.
- **C005 - Index & Navigation**: consults inbound links before deprecating or relocating a concept.

### C009 - Coverage Review

Supports periodically reviewing which areas of knowledge are represented against what the team actually relies on, so gaps in coverage surface instead of persisting unnoticed.

**Relationships**

- **C005 - Index & Navigation**: reads the directory of represented concepts to compare against expected coverage.
- **C010 - Scaffold**: uses the declared scope to frame what the reference is expected to cover.

### C010 - Scaffold

Lets the steward declare the reference's purpose and scope as an optional, persistent anchor that endures for the life of the reference; detects a scaffold edit that would change the immutable core purpose and routes the steward to fork-and-migrate rather than applying it in place; and supports periodically reconciling the graph against the declared scope to surface drifted content.

See [C010 - Scaffold](components/C010-scaffold.md).

## Requirement-Component Map

- **O001-R001 - Cited integration**: C003 - Integration Authoring
- **O001-R002 - Source revalidation**: C006 - Source Revalidation
- **O001-R003 - Captured-source retention**: C001 - Inbox
- **O002-R001 - Inbox intake**: C001 - Inbox
- **O002-R002 - Backlog visibility**: C001 - Inbox
- **O002-R003 - Coverage review**: C009 - Coverage Review
- **O003-R001 - Progressive index**: C005 - Index & Navigation
- **O003-R002 - Contextual cross-links**: C003 - Integration Authoring
- **O003-R003 - Naming conventions**: C004 - OKF Conformance Adapter
- **O003-R004 - Referential integrity on change**: C005 - Index & Navigation
- **O004-R001 - Timestamped concepts**: C007 - Currency Tracking
- **O004-R002 - Change history**: C007 - Currency Tracking
- **O005-R001 - Plain-text corpus**: C004 - OKF Conformance Adapter
- **O005-R002 - Open-format conformance**: C004 - OKF Conformance Adapter
- **O005-R003 - Version-control native**: C004 - OKF Conformance Adapter
- **O006-R001 - Deprecation over deletion**: C008 - Lifecycle & Retirement
- **O006-R002 - Retirement review**: C008 - Lifecycle & Retirement
- **O007-R001 - Agent-authored integration**: C003 - Integration Authoring
- **O007-R002 - Convention encapsulation**: C004 - OKF Conformance Adapter
- **O008-R001 - Scaffold declaration**: C010 - Scaffold
- **O008-R002 - Scope-aware triage**: C002 - Scope Triage
- **O008-R003 - Scope reconciliation**: C010 - Scaffold
