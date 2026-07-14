# Wiki Curation Architecture

Wiki is organized as a **curation pipeline over a shared corpus**, with format knowledge concentrated in one component. Raw material enters at one producer-agnostic point (the Inbox), is adjudicated against the reference's declared scope (Scope Triage), and — once the steward supplies intent — is authored into the corpus by the tool (Integration Authoring). Format, naming, and structural conventions live in exactly one component — **OKF Conformance** — as the conventions the agent authors to and a validator that checks the corpus, so no other component encodes format knowledge: the on-disk format is deferred entirely to the Open Knowledge Format, exactly as the product intends. Around this pipeline sit standing capabilities that keep the corpus discoverable, current, provenanced, on-scope, and high-signal (Index & Navigation, Source Revalidation, Currency Tracking, Lifecycle & Retirement, Coverage Review, Scaffold). Components read and write concepts directly, following the OKF conventions, and pass items along the pipeline; they do not share mutable state directly.

The steward drives the system through a curation **Agent Skill** exposing four operations — **Ingest**, **Query**, **Survey**, and **Lint** (see [ADR010 - Curation Agent Skill and Operations](drs/ADR010-curation-agent-skill.md)). A Wiki reference is an **OKF Knowledge Bundle** (SPEC §2/§3): a directory tree of markdown distributed as a git repository (recommended), a tarball, or a subdirectory. "Bundle" throughout Wiki means this.

## Principles

- **One owner of format knowledge.** Format, naming, and structural conventions live in exactly one component — OKF Conformance — as the conventions the agent authors to and a validator that checks the corpus. The rest of the system reasons about concepts and authors them directly following those conventions, rather than routing every corpus access through a runtime gateway (see [ADR007 - Conformance as Conventions and Validation](drs/ADR007-conformance-conventions-and-validation.md), which supersedes ADR001).
- **Intent in, authoring out.** Steward-facing components capture judgment and direction (keep, merge, deprecate, discard); tool-facing components perform all authoring, linking, and metadata. No component requires the steward to hand-edit the corpus.
- **Producer-agnostic intake.** The Inbox is the sole entry point and makes no assumption about who deposited an item, so automated producers, humans, and agents consuming the reference share one path and one adjudication gate.
- **Provenance and recency travel with the concept.** A statement's originating source and its last-meaningful-change time are first-class in-corpus metadata on the concept, not external bookkeeping (and not VCS history), so a consumer can check both at the moment of reliance wherever the bundle travels.
- **Never dead-end the graph.** Removal is modeled as deprecation with a replacement pointer, and any relocation or removal first resolves inbound links, so traversal never reaches nothing.
- **The scaffold is the single scope authority.** Both per-item triage and periodic reconciliation adjudicate against the one declared scaffold; its core purpose is revisable in place, and a revision triggers a Survey-driven reconciliation of the corpus against the new purpose (see [PDR004 - Revisable Core Purpose](../product/drs/PDR004-revisable-core-purpose.md), [ADR011 - In-Place Core-Purpose Change](drs/ADR011-in-place-core-purpose-change.md)).

## Constraints

- The corpus must remain plain-text and version-control-native at all times; no component may introduce a **proprietary or tool-locked** store. Curated concepts are plain-text markdown; a non-text *originating source* may be retained as a version-controlled binary asset for provenance (see [PDR002 - Provenance Assets Exception](../product/drs/PDR002-provenance-assets-exception.md) and [ADR008 - Captured Source Assets](drs/ADR008-captured-source-assets.md)).
- Wiki does not define or version the knowledge format. It consumes a published, versioned OKF; a format change is an OKF change, not a Wiki change.
- The scaffold's core purpose is revisable in place (see [PDR004 - Revisable Core Purpose](../product/drs/PDR004-revisable-core-purpose.md), superseding [PDR001 - Immutable Core Telos](../product/drs/PDR001-immutable-core-telos.md)). A change of purpose is applied in place and triggers a Survey-driven reconciliation of the corpus against the new purpose; version control preserves the prior whole-bundle state, so no fork and no cross-bundle relationships are needed.
- Integration is never automatic. An item reaches the corpus only after the steward's keep or merge direction; out-of-scope or unadjudicated items wait for judgment.
- Wiki targets a **low-volume, single-steward operating envelope**: every inbox item is triaged and every merge reconciled by hand, and the steward's adjudication capacity is the system's backpressure. Machine-volume unattended production is out of scope for the current design (see [ADR009 - Low-Volume Single-Steward Operating Envelope](drs/ADR009-low-volume-operating-envelope.md)).

## Technology Choices

- Git-tracked plain-text files as the corpus substrate: satisfies plain-text and version-control-native storage directly and needs no bespoke persistence layer.
- The Open Knowledge Format (OKF) as the storage and structural spec: the format is a product non-goal for Wiki to define, so the architecture depends on OKF rather than inventing a format. The version Wiki builds against is pinned in-repo at [`vendor/okf/SPEC.md`](../../vendor/okf/SPEC.md) (OKF 0.1; provenance in [`vendor/okf/PROVENANCE.md`](../../vendor/okf/PROVENANCE.md)); C004 - OKF Conformance is the only component that reads it, owning the conventions the agent authors to and the validator that checks the corpus against it.
- Agent-driven authoring, delivered as a curation **Agent Skill** (see [ADR010 - Curation Agent Skill and Operations](drs/ADR010-curation-agent-skill.md)): the steward works by intent while the agent performs authoring, linking, and metadata, keeping hands-on curation effort flat as material scales. The Skill exposes four operations — **Ingest** (capture → scope triage → integration authoring), **Query** (navigate the corpus and answer with citations; a valuable answer may be filed back into the Inbox as a producer deposit, so exploration compounds, without Wiki owning a query surface), **Survey** (the scaffold-relative assessment — measure the corpus against what the scaffold says it is *for*: coverage gaps, whose output is a **sourcing agenda** the steward acts on by finding sources that then re-enter through Ingest, and scope drift, i.e. content that has drifted out of the declared purpose), and **Lint** (the scaffold-independent hygiene pass: conformance validation, referential integrity, source revalidation, recency/staleness, and retirement review). Survey is where the scaffold's coverage differentiator is invoked; Lint corrects what already exists. Fuzzy judgments the framework already acknowledges — scope adjudication (C002) and core-vs-periphery detection (C010) — are realized as Skill prompt criteria anchored on the declared scaffold, not fixed algorithms.

## Components

### C001 - Inbox

The single, producer-agnostic capture point where raw material is recorded — with its origin — the moment it arrives, retained as durable provenance after processing, and where the count and age of un-integrated items are exposed.

See [C001 - Inbox](components/C001-inbox.md).

### C002 - Scope Triage

Adjudicates each inbox item against the declared scaffold scope, capturing the steward's keep, merge, deprecate, or discard direction and flagging out-of-scope items for judgment rather than integrating them automatically; when an item overlaps an existing concept, surfaces the overlapping concept(s) so a merge can be steered against a concrete target.

See [C002 - Scope Triage](components/C002-scope-triage.md).

### C003 - Integration Authoring

Turns a triaged item plus steward intent into corpus content — composing the concept, attaching its source link, and cross-linking related concepts with each reason stated — and, when an item overlaps an existing concept, executes the steward-chosen reconciliation rather than authoring blindly.

See [C003 - Integration Authoring](components/C003-integration-authoring.md).

### C004 - OKF Conformance

The single owner of OKF format knowledge: encapsulates the OKF authoring conventions the Agent Skill applies and provides a conformance validator that checks the bundle against OKF §9, so no other component and no contributor needs to know the format's conventions and no corpus access routes through a runtime gateway.

See [C004 - OKF Conformance](components/C004-okf-conformance.md).

### C005 - Index & Navigation

Maintains the navigable directory listing of what exists and traverses the cross-link graph, and surfaces the inbound links to a concept before it is removed or relocated so no consumer is left following a dangling reference.

See [C005 - Index & Navigation](components/C005-index-and-navigation.md).

### C006 - Source Revalidation

Detects when a cited source has changed since a statement was captured, by re-fetching the source and diffing it against the captured-source snapshot, so a curator can tell which statements no longer match the source they cite.

See [C006 - Source Revalidation](components/C006-source-revalidation.md).

### C007 - Currency Tracking

Records when each concept last meaningfully changed and maintains a dated, chronological history of updates for each scope of the reference, materialized as in-corpus OKF artifacts (`timestamp` and `log.md`) rather than VCS history, so recency and change history are legible to any consumer wherever the bundle travels.

See [C007 - Currency Tracking](components/C007-currency-tracking.md).

### C008 - Lifecycle & Retirement

Marks a concept as deprecated or superseded — with a reason and, where one exists, a pointer to its replacement — rather than hard-deleting it, and supports periodically identifying retirement candidates from measurable signals, so obsolete knowledge is retired without dead-ending the graph.

See [C008 - Lifecycle & Retirement](components/C008-lifecycle-and-retirement.md).

### C009 - Coverage Review

Supports periodically reviewing which areas of knowledge are represented against what the team actually relies on — assembling expected coverage from dangling links, the declared scaffold scope, and agent-proposed gaps — so gaps surface instead of persisting unnoticed.

See [C009 - Coverage Review](components/C009-coverage-review.md).

### C010 - Scaffold

Lets the steward declare the reference's purpose and scope as an optional, persistent anchor that endures for the life of the reference; applies a core-purpose revision in place and initiates a Survey-driven reconciliation of the corpus against the new purpose; and supports periodically reconciling the graph against the declared scope to surface drifted content.

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
- **O003-R003 - Naming conventions**: C004 - OKF Conformance
- **O003-R004 - Referential integrity on change**: C005 - Index & Navigation
- **O004-R001 - Timestamped concepts**: C007 - Currency Tracking
- **O004-R002 - Change history**: C007 - Currency Tracking
- **O005-R001 - Plain-text corpus**: C004 - OKF Conformance
- **O005-R002 - Open-format conformance**: C004 - OKF Conformance
- **O005-R003 - Version-control native**: C004 - OKF Conformance
- **O006-R001 - Deprecation over deletion**: C008 - Lifecycle & Retirement
- **O006-R002 - Retirement review**: C008 - Lifecycle & Retirement
- **O007-R001 - Agent-authored integration**: C003 - Integration Authoring
- **O007-R002 - Convention encapsulation**: C004 - OKF Conformance
- **O008-R001 - Scaffold declaration**: C010 - Scaffold
- **O008-R002 - Scope-aware triage**: C002 - Scope Triage
- **O008-R003 - Scope reconciliation**: C010 - Scaffold

## See Also

### Architectural Decision Records

- [ADR007 - Conformance as Conventions and Validation](drs/ADR007-conformance-conventions-and-validation.md)
- [ADR009 - Low-Volume Single-Steward Operating Envelope](drs/ADR009-low-volume-operating-envelope.md)
- [ADR010 - Curation Agent Skill and Operations](drs/ADR010-curation-agent-skill.md)
- [ADR011 - In-Place Core-Purpose Change](drs/ADR011-in-place-core-purpose-change.md)

### Change Records

- [CR003 - In-Corpus Currency](../crs/CR003-in-corpus-currency.md)
- [CR004 - Captured Source Assets](../crs/CR004-captured-source-assets.md)
- [CR005 - Conformance Boundary Re-scope](../crs/CR005-conformance-boundary-rescope.md)
- [CR007 - Low-Volume Single-Steward Operating Envelope](../crs/CR007-low-volume-operating-envelope.md)
- [CR008 - Delivery Form and Standing-Capability Specifications](../crs/CR008-delivery-form-and-standing-capabilities.md)
- [CR009 - Lifecycle & Retirement Component](../crs/CR009-lifecycle-retirement-component.md)
- [CR011 - Revisable Core Purpose](../crs/CR011-revisable-core-purpose.md)
