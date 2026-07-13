# Wiki — a curation discipline for shared knowledge

Wiki is hired by a **team or organization knowledge steward** to turn raw material into a trustworthy, navigable, portable shared reference that both teammates and agents can rely on. Where a Zettelkasten is a *personal* tool for **synthesizing** ideas into one's own thinking, Wiki is a *shared* discipline for **curating** what a group knows so others can consume it. It owns only the curation workflow — raw input → an inbox → integration into a knowledge corpus; the on-disk *format* is deferred entirely to the Open Knowledge Format specification (OKF).

The steward works by **intent and judgment, never by hand**: they feed raw material into an inbox and direct what happens to it — keep, merge, deprecate, discard — while the tool performs all authoring, linking, and metadata and keeps the corpus OKF-conformant. Because the inbox is **producer-agnostic**, automated producers (including OKF's own reference agent) can deposit material for the steward to adjudicate; Wiki governs the integration, it does not monopolize production.

A reference that knows its purpose can declare it in an optional **scaffold** — normally up front, or retrofitted once a maturing reference discovers its boundary. The scaffold is a persistent anchor the steward articulates and the tool maintains for the life of the reference; it ranges from a single statement of intent to a fuller set of boundaries and context, and it keeps an otherwise emergent knowledge graph tethered to the end it serves. Its core purpose is fixed — redefining that purpose constitutes a new reference, to which existing knowledge may be migrated, rather than an edit.

Wiki composes with a Zettelkasten rather than competing with it: **Wiki curates** the shared, cited source-of-record, and a Zettelkasten in the same project **consumes Wiki as source material and synthesizes** it into personal thinking notes — a *curate → synthesize* pipeline.

Non-goals: Wiki is not an unattended enrichment pipeline for a single source system (that is what OKF's reference agent demonstrates), not a personal thinking tool (that is the Zettelkasten), and not a definition of the storage format (that is OKF).

## Jobs

### Knowledge Stewardship

> When raw material my team will later depend on accumulates faster than anyone can absorb it, I want to turn it into a trustworthy, navigable, portable shared reference that stays focused on what it is for, so any teammate or agent can find and rely on what we know without routing every question through me.

#### O001 - Trustworthy provenance

Maximize the proportion of statements in the reference that trace to a verifiable source at the moment a consumer relies on them.

**Risks**

- **O001-RSK001** - Unsourced capture: Material is integrated without its origin recorded, so a consumer cannot confirm where a claim came from and must re-verify it independently.
- **O001-RSK002** - Source drift: An underlying source changes or disappears after capture, so a recorded statement no longer matches the source it cites.

**Requirements**

- **O001-R001** - Cited integration: The product must require every integrated statement to carry a link to its originating source.
- **O001-R002** - Source revalidation: The product must let a curator detect when a cited source has changed since the statement was captured.

**Risk-Requirement Map**

- **O001-RSK001 - Unsourced capture**: O001-R001 - Cited integration
- **O001-RSK002 - Source drift**: O001-R002 - Source revalidation

#### O002 - Complete coverage

Minimize the likelihood that knowledge the team relies on is absent from the reference.

**Risks**

- **O002-RSK001** - Pre-integration loss: Raw material is discarded or lost before it is ever triaged, so knowledge the team relies on never enters the reference at all.
- **O002-RSK002** - Silent scope gaps: No signal reveals that an entire area of the team's knowledge was never captured, so the gap persists unnoticed.
- **O002-RSK003** - Untracked backlog: Captured material sits un-integrated with no visibility into how much is waiting or how old it is, so it silently never reaches consumers.

**Requirements**

- **O002-R001** - Inbox intake: The product must provide a single inbox where raw material is captured the moment it arrives, so nothing relied upon is lost before it can be triaged.
- **O002-R002** - Backlog visibility: The product must make the count and age of un-integrated inbox items visible so a growing backlog cannot go unnoticed.
- **O002-R003** - Coverage review: The product must support periodically reviewing which areas of knowledge are represented against what the team actually relies on.

**Risk-Requirement Map**

- **O002-RSK001 - Pre-integration loss**: O002-R001 - Inbox intake
- **O002-RSK002 - Silent scope gaps**: O002-R003 - Coverage review
- **O002-RSK003 - Untracked backlog**: O002-R002 - Backlog visibility

#### O003 - Fast discovery

Minimize the time to locate the knowledge relevant to a question when navigating the reference.

**Risks**

- **O003-RSK001** - Flat sprawl: The reference grows without navigational structure, so a consumer must scan the whole corpus to find what is relevant.
- **O003-RSK002** - Ambiguous naming: Concepts are titled and identified inconsistently, so a searcher cannot tell which document answers their question.
- **O003-RSK003** - Broken navigation on removal: When a concept is removed or relocated, inbound links are left pointing at nothing, so a consumer traversing relationships reaches a dead end instead of the related knowledge.

**Requirements**

- **O003-R001** - Progressive index: The product must maintain a navigable directory listing so a consumer can see what exists before opening individual documents.
- **O003-R002** - Contextual cross-links: The product must link related concepts, with the reason for each link stated, so consumers can traverse relationships rather than only search.
- **O003-R003** - Naming conventions: The product must enforce consistent, descriptive titles and stable identifiers for concepts.
- **O003-R004** - Referential integrity on change: The product must surface inbound links to a concept before it is removed or relocated, so consumers are not left following dangling references.

**Risk-Requirement Map**

- **O003-RSK001 - Flat sprawl**: O003-R001 - Progressive index, O003-R002 - Contextual cross-links
- **O003-RSK002 - Ambiguous naming**: O003-R003 - Naming conventions
- **O003-RSK003 - Broken navigation on removal**: O003-R004 - Referential integrity on change

#### O004 - Currency

Minimize the lag between a change in an underlying source and the reference reflecting that change.

**Risks**

- **O004-RSK001** - Undetected staleness: An underlying source changes with no signal in the reference, so consumers rely on outdated knowledge believing it is current.
- **O004-RSK002** - Unclear recency: A consumer cannot tell how old a statement is, so no one knows when it is due for reconciliation.

**Requirements**

- **O004-R001** - Timestamped concepts: The product must record when each concept last meaningfully changed.
- **O004-R002** - Change history: The product must record a dated, chronological history of updates for each scope of the reference.

**Risk-Requirement Map**

- **O004-RSK001 - Undetected staleness**: O001-R002 - Source revalidation
- **O004-RSK002 - Unclear recency**: O004-R001 - Timestamped concepts, O004-R002 - Change history

#### O005 - Portable reuse

Minimize the effort required to move or reuse the accumulated knowledge in a different tool, team, or organization.

**Risks**

- **O005-RSK001** - Tool lock-in: Knowledge is captured in a form tied to one tool or service, so reusing it elsewhere requires exporting and reworking it.
- **O005-RSK002** - Non-self-describing corpus: The knowledge lacks the structural conventions needed to be understood outside its origin, so moving it requires someone to re-explain it.

**Requirements**

- **O005-R001** - Plain-text corpus: The product must store knowledge as plain, human-readable files that require no proprietary tool to read.
- **O005-R002** - Open-format conformance: The product must keep the corpus conformant to a published, versioned open knowledge format so it is interpretable outside its origin.
- **O005-R003** - Version-control native: The product must keep the corpus diffable and mergeable in standard version control.

**Risk-Requirement Map**

- **O005-RSK001 - Tool lock-in**: O005-R001 - Plain-text corpus, O005-R003 - Version-control native
- **O005-RSK002 - Non-self-describing corpus**: O005-R002 - Open-format conformance

#### O006 - High signal

Minimize the proportion of the knowledge a consumer encounters that is obsolete, superseded, or no longer relied upon.

**Risks**

- **O006-RSK001** - Accretion without removal: Knowledge is only ever added and never retired, so obsolete concepts accumulate and dilute what is still current.
- **O006-RSK002** - Unmarked supersession: When a concept is replaced, the outdated version is left without a clear deprecation marker, so consumers cannot tell it is superseded and keep relying on the stale version.

**Requirements**

- **O006-R001** - Deprecation over deletion: The product must support marking a concept as deprecated or superseded — with a reason and, where one exists, a pointer to its replacement — rather than only hard-deleting it.
- **O006-R002** - Retirement review: The product must support periodically identifying concepts no longer relied upon as candidates for pruning.

**Risk-Requirement Map**

- **O006-RSK001 - Accretion without removal**: O006-R001 - Deprecation over deletion, O006-R002 - Retirement review
- **O006-RSK002 - Unmarked supersession**: O006-R001 - Deprecation over deletion

#### O007 - Low curation effort

Minimize the hands-on effort a steward must spend to turn a unit of raw material into curated, consumable knowledge.

**Risks**

- **O007-RSK001** - Mechanical toil: Turning raw material into conformant knowledge requires manual formatting, linking, and metadata entry, so the effort to keep the reference current scales with the volume of material rather than staying flat.
- **O007-RSK002** - Expertise bottleneck: Only someone fluent in the format's conventions can add to the reference, so curation stalls whenever that person is unavailable.

**Requirements**

- **O007-R001** - Agent-authored integration: The product must let the steward integrate material by intent — supplying the raw input and keep, merge, deprecate, or discard direction — while the product itself performs the authoring, linking, and metadata.
- **O007-R002** - Convention encapsulation: The product must apply the format's structural and naming conventions automatically, so a contributor need not know them to add conformant knowledge.

**Risk-Requirement Map**

- **O007-RSK001 - Mechanical toil**: O007-R001 - Agent-authored integration, O007-R002 - Convention encapsulation
- **O007-RSK002 - Expertise bottleneck**: O007-R002 - Convention encapsulation

#### O008 - On-scope focus

Minimize the proportion of the reference that falls outside its intended purpose and scope.

**Risks**

- **O008-RSK001** - Input-driven drift: The reference expands to whatever material arrives rather than what its purpose calls for, so content that was never on-mission takes up residence and blurs what the reference is for.
- **O008-RSK002** - Implicit boundary: The purpose and scope are held only in a steward's head rather than stated, so neither a contributor nor a consumer can adjudicate what belongs and drift goes unnoticed.

**Requirements**

- **O008-R001** - Scaffold declaration: The product must let a steward declare the reference's purpose and scope as an optional, persistent anchor — from a single statement of intent to a fuller set of boundaries and context — that endures for the life of the reference.
- **O008-R002** - Scope-aware triage: The product must evaluate incoming material against the declared scope and flag out-of-scope items for the steward's judgment rather than integrating them automatically.
- **O008-R003** - Scope reconciliation: The product must support periodically reconciling the graph against the declared scope, surfacing content that has drifted out of scope.

**Risk-Requirement Map**

- **O008-RSK001 - Input-driven drift**: O008-R002 - Scope-aware triage, O008-R003 - Scope reconciliation
- **O008-RSK002 - Implicit boundary**: O008-R001 - Scaffold declaration

**Product Decision Records**

- [PDR001 - Immutable Core Telos](drs/PDR001-immutable-core-telos.md)
