# Wiki — a curation tool for shared knowledge

Wiki is hired by a **knowledge steward** to turn the raw material a team accumulates into a shared body of knowledge any teammate or agent can find and rely on — without routing every question through the one person who holds it in their head. Success is knowledge that stays trustworthy as it grows: traceable to its source, current, complete for what it is meant to cover, and portable enough to outlive any one tool.

## Jobs

### Knowledge Stewardship

> When raw material my team will come to rely on accumulates faster than anyone can absorb it, I want to turn it into a shared body of knowledge that stays trustworthy as it grows, so any teammate or agent can find and rely on what we know without routing every question through me.

#### O001 - Traceable provenance

Maximize the proportion of claims in the body of knowledge a consumer can trace to a verifiable source at the moment they rely on them.

**Risks**

- **O001-RSK001** - Uncited claim: A claim enters the body of knowledge with no link to the source it was drawn from, so a consumer cannot confirm where it came from and must re-verify it independently.
- **O001-RSK002** - Source rot: A cited source is referenced but not durably held, so when it later changes or disappears the claim can no longer be checked against the form it was drawn from.

**Requirements**

- **O001-R001** - Cited claims: The product must give every claim it writes a reference to the originating source it was drawn from.
- **O001-R002** - Durable source reference: The product must ensure each source reference resolves to a form version-controlled alongside the body of knowledge, capturing a snapshot of the source only when it cannot be referenced durably in place.
- **O001-R003** - Drift detection: The product must let the steward detect when a referenced source has changed since the claim was drawn from it.

**Risk-Requirement Map**

- **O001-RSK001 - Uncited claim**: O001-R001 - Cited claims
- **O001-RSK002 - Source rot**: O001-R002 - Durable source reference, O001-R003 - Drift detection

#### O002 - Currency

Minimize the lag between a change in an underlying source and the body of knowledge reflecting that change.

**Risks**

- **O002-RSK001** - Undetected staleness: An underlying source changes with no signal in the body of knowledge, so consumers keep relying on a claim that has quietly gone out of date.
- **O002-RSK002** - Unclear recency: A consumer cannot tell how recently a claim was checked or changed, so no one can judge when it is due for review.

**Requirements**

- **O002-R001** - Recorded recency: The product must record when each concept last meaningfully changed, legibly to any consumer of the body of knowledge.
- **O002-R002** - Change history: The product must maintain a dated history of changes for each part of the body of knowledge.

**Risk-Requirement Map**

- **O002-RSK001 - Undetected staleness**: O001-R003 - Drift detection, O002-R001 - Recorded recency
- **O002-RSK002 - Unclear recency**: O002-R001 - Recorded recency, O002-R002 - Change history

#### O003 - Sustained signal

Minimize the proportion of the knowledge a consumer encounters that is low-signal — obsolete, superseded, redundant, insubstantial, or no longer relied upon.

**Risks**

- **O003-RSK001** - Accretion without removal: Knowledge is only ever added and never retired, so obsolete concepts accumulate and dilute what is still current.
- **O003-RSK002** - Unmarked supersession: A concept is replaced but the superseded version is left unmarked, so consumers cannot tell it is stale and keep relying on it.
- **O003-RSK003** - Redundant capture: The same knowledge enters more than once, so duplicate concepts accumulate and a consumer cannot tell which to rely on.
- **O003-RSK004** - Low-value accretion: Trivial or insubstantial material is integrated, so the body of knowledge fills with noise that dilutes what is worth relying on.

**Requirements**

- **O003-R001** - Deprecation over deletion: The product must support marking a concept deprecated or superseded — with a reason and, where one exists, a pointer to its replacement — rather than only removing it.
- **O003-R002** - Retirement review: The product must periodically surface retirement candidates from measurable signals — staleness, source drift, and supersession — for the steward's judgment of what is still relied upon.
- **O003-R003** - Duplication check: The product must detect when incoming material overlaps an existing concept and surface the overlap for the steward to merge, keep as new, or discard, rather than admitting a blind duplicate.
- **O003-R004** - Significance bar: The product must assess whether incoming material is substantial enough to keep and flag insubstantial material for the steward rather than integrating it automatically.

**Risk-Requirement Map**

- **O003-RSK001 - Accretion without removal**: O003-R001 - Deprecation over deletion, O003-R002 - Retirement review
- **O003-RSK002 - Unmarked supersession**: O003-R001 - Deprecation over deletion
- **O003-RSK003 - Redundant capture**: O003-R003 - Duplication check
- **O003-RSK004 - Low-value accretion**: O003-R004 - Significance bar

#### O004 - Low curation effort

Minimize the hands-on effort required of the steward to keep the body of knowledge trustworthy and current as material accumulates.

**Risks**

- **O004-RSK001** - Mechanical toil: Turning raw material into conformant knowledge requires manual formatting, linking, and metadata, so the effort to add knowledge scales with the volume of material rather than staying flat.
- **O004-RSK002** - Expertise bottleneck: Only someone fluent in the format's conventions can add to the body of knowledge, so curation stalls whenever that person is unavailable.
- **O004-RSK003** - Manual upkeep: Keeping the body of knowledge current and pruned requires the steward to audit the whole corpus by hand, so maintenance effort grows with its size.

**Requirements**

- **O004-R001** - Intent-driven authoring: The product must let the steward turn raw material into knowledge by giving direction, while the product performs the authoring, linking, and metadata.
- **O004-R002** - Convention encapsulation: The product must apply the format's structural and naming conventions automatically, so a contributor need not know them to add conformant knowledge.
- **O004-R003** - Assisted upkeep: The product must surface what needs the steward's attention and carry out the resulting changes by intent, so maintenance requires no hand-auditing or hand-editing of the corpus.

**Risk-Requirement Map**

- **O004-RSK001 - Mechanical toil**: O004-R001 - Intent-driven authoring, O004-R002 - Convention encapsulation
- **O004-RSK002 - Expertise bottleneck**: O004-R002 - Convention encapsulation
- **O004-RSK003 - Manual upkeep**: O004-R003 - Assisted upkeep

#### O005 - Fast discovery

Minimize the time to locate the knowledge relevant to a question when navigating the body of knowledge.

**Risks**

- **O005-RSK001** - Flat sprawl: The body of knowledge grows without navigational structure, so a consumer must scan the whole of it to find what is relevant.
- **O005-RSK002** - Dead-end on change: When a concept is retired or relocated, links pointing to it are left dangling, so a consumer traversing relationships reaches nothing.

**Requirements**

- **O005-R001** - Navigable index: The product must maintain a navigable listing of what the body of knowledge contains, so a consumer can see what exists before opening individual documents.
- **O005-R002** - Reasoned cross-links: The product must link related concepts, with the reason for each link stated, so consumers can traverse relationships rather than only search.
- **O005-R003** - Referential integrity: The product must surface the inbound links to a concept before it is retired or relocated, so they can be resolved rather than left dangling.

**Risk-Requirement Map**

- **O005-RSK001 - Flat sprawl**: O005-R001 - Navigable index, O005-R002 - Reasoned cross-links
- **O005-RSK002 - Dead-end on change**: O005-R003 - Referential integrity

#### O006 - Portable reuse

Minimize the effort required to move or reuse the accumulated knowledge in a different tool, team, or organization.

**Risks**

- **O006-RSK001** - Tool lock-in: Knowledge is held in a form tied to one tool or service, so reusing it elsewhere requires exporting and reworking it.
- **O006-RSK002** - Non-self-describing corpus: The knowledge lacks the structural conventions needed to be understood outside its origin, so moving it requires someone to re-explain it.

**Requirements**

- **O006-R001** - Plain-text corpus: The product must store the body of knowledge as plain-text files readable with no proprietary tool.
- **O006-R002** - Open-format conformance: The product must keep the body of knowledge conformant to a published, versioned open knowledge format so it is interpretable outside its origin.
- **O006-R003** - Version-control-native: The product must keep the body of knowledge diffable and mergeable in standard version control.

**Risk-Requirement Map**

- **O006-RSK001 - Tool lock-in**: O006-R001 - Plain-text corpus, O006-R003 - Version-control-native
- **O006-RSK002 - Non-self-describing corpus**: O006-R002 - Open-format conformance

#### O007 - Complete coverage

Minimize the likelihood that knowledge within the body of knowledge's intended scope is absent from it.

**Risks**

- **O007-RSK001** - Lost before intake: Raw material is discarded or forgotten before it is ever triaged, so knowledge the team relies on never enters the body of knowledge at all.
- **O007-RSK002** - Silent gaps: No signal reveals that an area within the intended scope was never captured, so the gap persists unnoticed.

**Requirements**

- **O007-R001** - Producer-agnostic intake: The product must provide a single intake point where any producer can submit raw material the moment it arrives, so nothing relied upon is lost before it can be triaged.
- **O007-R002** - Coverage review: The product must periodically surface areas within the intended scope that are thin or absent, for the steward's judgment of what the team relies on.

**Risk-Requirement Map**

- **O007-RSK001 - Lost before intake**: O007-R001 - Producer-agnostic intake
- **O007-RSK002 - Silent gaps**: O007-R002 - Coverage review

#### O008 - On-scope focus

Minimize the proportion of the body of knowledge that falls outside its intended scope.

**Risks**

- **O008-RSK001** - Input-driven drift: The body of knowledge expands to whatever material arrives rather than what its purpose calls for, so off-purpose content takes up residence and blurs what it is for.
- **O008-RSK002** - Implicit boundary: The intended scope lives only in the steward's head rather than being stated, so no one can adjudicate what belongs and drift goes unnoticed.

**Requirements**

- **O008-R001** - Charter declaration: The product must let the steward declare the body of knowledge's purpose and scope as an optional, persistent anchor — the charter — that endures for its life and is revisable in place.
- **O008-R002** - Scope-aware triage: The product must evaluate incoming material against the declared scope and flag out-of-scope material for the steward rather than admitting it automatically.
- **O008-R003** - Scope reconciliation: The product must periodically surface content that has drifted outside the declared scope.

**Risk-Requirement Map**

- **O008-RSK001 - Input-driven drift**: O008-R002 - Scope-aware triage, O008-R003 - Scope reconciliation
- **O008-RSK002 - Implicit boundary**: O008-R001 - Charter declaration
