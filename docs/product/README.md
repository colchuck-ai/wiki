# Wiki — a curation tool for shared knowledge

Wiki is hired by a **knowledge steward** to turn the raw material a team
accumulates into a **body of knowledge** — a curated set of cited concepts.
That body of knowledge is a product in its own right: it is in turn *hired by
consumers* — teammates and agents — to answer their questions without routing
through the one person who holds it in their head.

So there are two hires. The steward hires **Wiki** (the tool) to curate. The
consumer hires **the body of knowledge** (the deliverable) to find and rely on
what the team knows. The consumer never touches Wiki. Wiki succeeds when the
corpus it produces is one those consumers can hire and trust on sight:

> **traceable** to its source, **faithful** to what that source says,
> **current**, **high-signal**, **discoverable**, **on-scope**, **complete**
> for what it is meant to cover, and **portable** enough to outlive any one
> tool — produced at a curation effort that stays flat as the material grows.

The first eight properties are qualities of the *deliverable*, owed to the
consumer. The last is a quality of the *steward's own experience* using Wiki.
The tree keeps that distinction explicit (see **Outcome families** below).

## Actors

- **Steward** — hires Wiki; supplies curation *judgment* (declares the charter,
  sets the delegation envelope, adjudicates escalated exceptions). Wiki's user.
- **Consumer** — hires the *deliverable*, not Wiki; a **teammate or agent**
  whose success anchors the deliverable-quality outcomes but is not itself
  measured on Wiki (Wiki does not control the consumer's question, skill, or
  reader).
- **Producer** — submits raw material at intake; need not be the steward.

## The deliverable and its downstream job

Wiki's deliverable is the **body of knowledge**. It is itself hired — by a
**consumer** — for a job Wiki does not perform but must *enable*:

> When I have a question my team should already know the answer to, I want to
> find it and trust it on sight, so I can act without interrupting the person
> who holds it in their head.

This downstream job is **not measured on Wiki**: Wiki controls the corpus, not
the consumer's session. It is the *anchor* from which the deliverable-quality
outcomes below derive their existence and their metrics. When an outcome speaks
of what "a consumer can trace" or "time to locate," it is measured against
*this* job — which is why each such outcome also states an observable **proxy**
(see Operating principles).

## Foundational concepts

The outcomes presuppose these. They are corpus-level nouns, not features of any
one outcome.

- **Charter** — the corpus's declared **purpose and scope**. It is
  **required** (a curated corpus has a purpose the way it has an identity),
  **revisable in place** (provisional at first, sharpened as the material is
  understood — required to *exist*, not to be complete on day one), and
  **load-bearing**: triage (O008), the significance bar (O003), duplication
  (O003), and coverage (O007) all read against it. A required-but-unconsulted
  charter would be dead compliance; it earns "required" by being an input to
  those decisions. A consumer can read it to know what the corpus is *for*.
- **Delegation envelope** — the steward's declaration of what Wiki may do
  **autonomously** and what it must **escalate**. Optional, and *tuned* as the
  steward's trust in Wiki grows.
- **Policy anchor** — the pattern both of the above share: a steward-declared,
  persistent, in-corpus, revisable-in-place statement governing Wiki's
  behavior. They are two distinct anchors (different audiences — the charter is
  consumer-legible, the envelope is operational; different change cadence; and
  the envelope *reads* the charter), sharing one treatment of declaration,
  persistence, and revision. They differ only in that the charter is required
  and the envelope optional.
- **Decision-provenance & review tier** — because Wiki acts autonomously within
  the envelope, the corpus will hold claims no human gated. That is safe only
  if every item records *how it was handled* (**decision-provenance**: outcome,
  actor, envelope version, time) and consumers can see *who vetted it* (a
  **review tier**: human-reviewed vs. autonomously integrated). Autonomy is
  trustworthy because it is legible and reversible, not because it is trusted
  blindly.
- **Body of knowledge as a structured substrate** — one set of plain-text,
  open-format files serving two access modes: a human view and an
  agent-parseable form. See *machine-readable-first* below.

## Operating principles

Invariants the outcomes and concepts inherit; stated once here.

- **Manage by exception.** Wiki acts autonomously within the delegation
  envelope and escalates only genuinely ambiguous cases. This is why the
  admission and upkeep requirements read "act within the envelope, escalate
  exceptions" rather than "flag everything for the steward" — a synchronous
  human gate on every item would relocate the very bottleneck the product
  exists to remove (O004) from the reader to the writer.
- **Machine-readable-first.** The consumer is *a teammate or agent*. Everything
  legible to a human — grounding signal, review tier, recency, link reasons —
  is a machine-readable field first and a rendering second. A signal that lives
  only in prose is invisible to half the declared consumers.
- **Transferable stewardship.** The corpus should hold everything a steward
  needs to steward it, so curation judgment does not live in one head — the
  steward-side mirror of the product's core promise. The charter, envelope,
  decision-provenance, and reasoned cross-links each externalize a piece of
  that judgment into the corpus, so the role can be handed off by *reading*,
  not by downloading the last steward's memory.
- **Deprecation over deletion.** Knowledge and material are marked and kept
  (deprecated, superseded, held aside), never silently destroyed; genuine
  purging is an explicit, separate act. This applies at intake as much as at
  retirement.
- **Proxy-measured outcomes.** A deliverable-quality outcome states its
  consumer-value *true metric* (the goal, often unobservable from inside Wiki)
  *and* an observable **proxy** — a corpus property Wiki can measure that
  stands in for it. The proxy is what Wiki optimizes; the true metric is why.

## Outcome families

Each outcome is tagged with the family it serves:

- *deliverable-quality* — a property of the corpus, owed to the **consumer**:
  O001, O002, O003, O005, O006, O007, O008, O009.
- *curation-experience* — a property of the **steward's** experience operating
  Wiki: O004. (The lone outcome that would still matter if no consumer ever
  existed.)

## Jobs

### Knowledge Stewardship

> When raw material my team will come to rely on accumulates faster than anyone
> can absorb it, I want to turn it into a shared body of knowledge that stays
> trustworthy as it grows, so any teammate or agent can find and rely on what
> we know without routing every question through me.

#### O001 - Traceable provenance

*Family: deliverable-quality.*
**True metric:** maximize the proportion of claims a consumer can trace to a
verifiable source at the moment they rely on them.
**Proxy:** proportion of claims carrying a citation that resolves to a durable,
version-controlled form.

**Risks**

- **O001-RSK001** - Uncited claim: A claim enters the body of knowledge with no
  link to the source it was drawn from, so a consumer cannot confirm where it
  came from and must re-verify it independently.
- **O001-RSK002** - Source rot: A cited source is referenced but not durably
  held, so when it later changes or disappears the claim can no longer be
  checked against the form it was drawn from.

**Requirements**

- **O001-R001** - Cited claims: The product must give every claim it writes a
  reference to the originating source it was drawn from.
- **O001-R002** - Citation resolves to a durable form: The product must make
  each citation *resolve* to a durable form version-controlled alongside the
  body of knowledge — capturing a snapshot of the source when it cannot be
  referenced durably in place — so that traceability and fidelity check against
  a form that cannot rot. Where the source has a live external locator, that
  locator is carried as a *distinct* structured reference (for drift
  detection), never as a substitute for the durable resolution target.
- **O001-R003** - Drift detection: The product must let the steward detect when
  a referenced source has changed since the claim was drawn from it, by reading
  the live-origin locator against the durable snapshot.

**Risk-Requirement Map**

- **O001-RSK001 - Uncited claim**: O001-R001 - Cited claims
- **O001-RSK002 - Source rot**: O001-R002 - Citation resolves to a durable form, O001-R003 - Drift detection

#### O002 - Currency

*Family: deliverable-quality.*
**True metric:** minimize the lag between a change in an underlying source and
the body of knowledge reflecting that change.
**Proxy:** proportion of concepts whose recorded recency and drift status are
up to date relative to their sources.

**Risks**

- **O002-RSK001** - Undetected staleness: An underlying source changes with no
  signal in the body of knowledge, so consumers keep relying on a claim that
  has quietly gone out of date.
- **O002-RSK002** - Unclear recency: A consumer cannot tell how recently a
  claim was checked or changed, so no one can judge when it is due for review.

**Requirements**

- **O002-R001** - Recorded recency: The product must record when each concept
  last meaningfully changed, as a machine-readable field legible to any
  consumer of the body of knowledge.
- **O002-R002** - Change history: The product must maintain a dated history of
  changes for each part of the body of knowledge, including the
  decision-provenance of each change (outcome, actor, envelope version).

**Risk-Requirement Map**

- **O002-RSK001 - Undetected staleness**: O001-R003 - Drift detection, O002-R001 - Recorded recency
- **O002-RSK002 - Unclear recency**: O002-R001 - Recorded recency, O002-R002 - Change history

#### O003 - Sustained signal

*Family: deliverable-quality.*
**True metric:** minimize the proportion of the knowledge a consumer encounters
that is low-signal — obsolete, superseded, redundant, insubstantial, or no
longer relied upon.
**Proxy:** proportion of consumer-facing concepts flagged by retirement,
supersession, duplication, or significance signals and not yet resolved.

**Risks**

- **O003-RSK001** - Accretion without removal: Knowledge is only ever added and
  never retired, so obsolete concepts accumulate and dilute what is still
  current.
- **O003-RSK002** - Unmarked supersession: A concept is replaced but the
  superseded version is left unmarked, so consumers cannot tell it is stale and
  keep relying on it.
- **O003-RSK003** - Redundant capture: The same knowledge enters more than
  once, so duplicate concepts accumulate and a consumer cannot tell which to
  rely on.
- **O003-RSK004** - Low-value accretion: Trivial or insubstantial material is
  integrated, so the body of knowledge fills with noise that dilutes what is
  worth relying on.

**Requirements**

- **O003-R001** - Deprecation over deletion: The product must support marking a
  concept deprecated or superseded — with a reason and, where one exists, a
  pointer to its replacement — rather than only removing it.
- **O003-R002** - Retirement review: The product must periodically surface
  retirement candidates from measurable signals — staleness, source drift, and
  supersession — acting within the delegation envelope and escalating the calls
  it is not authorized to make for the steward's judgment of what is still
  relied upon.
- **O003-R003** - Duplication check: The product must detect when incoming
  material overlaps an existing concept and, within the delegation envelope,
  resolve the overlap (merge, keep as new, or discard) or escalate the
  ambiguous case — rather than admitting a blind duplicate. Overlap is judged
  relative to the charter's scope.
- **O003-R004** - Significance bar: The product must assess whether incoming
  material is substantial enough — relative to the charter — to enter the
  consumer-facing corpus, and route below-bar material to a recoverable
  **held-aside** state rather than discarding it. The bar gates
  *integrate-versus-hold-aside*, not *keep-versus-destroy*: a wrong call stays
  auditable and reversible, and genuine purging is a separate explicit act.

**Risk-Requirement Map**

- **O003-RSK001 - Accretion without removal**: O003-R001 - Deprecation over deletion, O003-R002 - Retirement review
- **O003-RSK002 - Unmarked supersession**: O003-R001 - Deprecation over deletion
- **O003-RSK003 - Redundant capture**: O003-R003 - Duplication check
- **O003-RSK004 - Low-value accretion**: O003-R004 - Significance bar

#### O004 - Low curation effort

*Family: curation-experience.*
**Metric:** minimize the hands-on effort required of the steward to keep the
body of knowledge trustworthy and current as material accumulates — effort that
stays flat as volume grows, whether the toil is of the *hands* (formatting) or
of the *eyes* (reviewing every item).

**Risks**

- **O004-RSK001** - Mechanical toil: Turning raw material into conformant
  knowledge requires manual formatting, linking, and metadata, so the effort to
  add knowledge scales with the volume of material rather than staying flat.
- **O004-RSK002** - Expertise bottleneck: Only someone fluent in the format's
  conventions can add to the body of knowledge, so curation stalls whenever
  that person is unavailable.
- **O004-RSK003** - Manual upkeep: Keeping the body of knowledge current and
  pruned requires the steward to audit the whole corpus by hand, so maintenance
  effort grows with its size.
- **O004-RSK004** - Review-gate toil: Requiring the steward to adjudicate every
  admission and upkeep action makes their effort scale with input volume — the
  reader's bottleneck relocated to the writer.

**Requirements**

- **O004-R001** - Intent-driven authoring: The product must let the steward
  turn raw material into knowledge by giving direction, while the product
  performs the authoring, linking, and metadata.
- **O004-R002** - Convention encapsulation: The product must apply the format's
  structural and naming conventions automatically, so a contributor need not
  know them to add conformant knowledge.
- **O004-R003** - Assisted upkeep: The product must surface what needs the
  steward's attention and carry out the resulting changes by intent, so
  maintenance requires no hand-auditing or hand-editing of the corpus.
- **O004-R004** - Delegated autonomy: The product must let the steward declare
  a delegation envelope and then act autonomously within it, escalating only
  exceptions, so routine admission and upkeep do not require per-item steward
  adjudication.

**Risk-Requirement Map**

- **O004-RSK001 - Mechanical toil**: O004-R001 - Intent-driven authoring, O004-R002 - Convention encapsulation
- **O004-RSK002 - Expertise bottleneck**: O004-R002 - Convention encapsulation
- **O004-RSK003 - Manual upkeep**: O004-R003 - Assisted upkeep
- **O004-RSK004 - Review-gate toil**: O004-R004 - Delegated autonomy

#### O005 - Fast discovery

*Family: deliverable-quality.*
**True metric:** minimize the time to locate the knowledge relevant to a
question — for a human *navigating* the corpus and for an agent *querying* it.
**Proxy:** index coverage over all concepts, cross-link density with stated
reasons, and absence of dangling links — exposed both as a rendered view and as
machine-parseable structure.

**Risks**

- **O005-RSK001** - Flat sprawl: The body of knowledge grows without
  navigational structure, so a consumer must scan the whole of it to find what
  is relevant.
- **O005-RSK002** - Dead-end on change: When a concept is retired or relocated,
  links pointing to it are left dangling, so a consumer traversing
  relationships reaches nothing.
- **O005-RSK003** - Human-only affordance: Discovery aids exist only as
  human-rendered views, so an agent consumer cannot find or traverse the corpus
  programmatically.

**Requirements**

- **O005-R001** - Navigable index: The product must maintain a navigable
  listing of what the body of knowledge contains — as machine-readable
  structure with a human-legible rendering — so a consumer can see what exists
  before opening individual documents.
- **O005-R002** - Reasoned cross-links: The product must link related concepts,
  with the reason for each link stated, so consumers can traverse relationships
  rather than only search.
- **O005-R003** - Referential integrity: The product must surface the inbound
  links to a concept before it is retired or relocated, so they can be resolved
  rather than left dangling.

**Risk-Requirement Map**

- **O005-RSK001 - Flat sprawl**: O005-R001 - Navigable index, O005-R002 - Reasoned cross-links
- **O005-RSK002 - Dead-end on change**: O005-R003 - Referential integrity
- **O005-RSK003 - Human-only affordance**: O005-R001 - Navigable index, O005-R002 - Reasoned cross-links

#### O006 - Portable reuse

*Family: deliverable-quality.*
**True metric:** minimize the effort required to move or reuse the accumulated
knowledge in a different tool, team, or organization.
**Proxy:** proportion of the corpus that is plain-text, version-controlled, and
validates against the pinned open-format spec.

Portability has two independent layers of different strength: **substrate
portability** (plain text + version control — unconditional, needs no one's
cooperation) and **semantic portability** (conformance to an external open
format — as reachable as that format is adopted; the format Wiki targets is an
early, single-origin draft, so this layer's practical reach grows with its
adoption rather than being assumed mature).

**Risks**

- **O006-RSK001** - Tool lock-in: Knowledge is held in a form tied to one tool
  or service, so reusing it elsewhere requires exporting and reworking it.
- **O006-RSK002** - Non-self-describing corpus: The knowledge lacks the
  structural conventions needed to be understood outside its origin, so moving
  it requires someone to re-explain it.

**Requirements**

- **O006-R001** - Plain-text corpus: The product must store the body of
  knowledge as plain-text files readable with no proprietary tool.
- **O006-R002** - Open-format conformance: The product must keep the body of
  knowledge conformant to an *externally* published, versioned open knowledge
  format — one Wiki consumes rather than defines — so it is interpretable
  outside its origin by anyone with the spec, not only by Wiki.
- **O006-R003** - Version-control-native: The product must keep the body of
  knowledge diffable and mergeable in standard version control.

**Risk-Requirement Map**

- **O006-RSK001 - Tool lock-in**: O006-R001 - Plain-text corpus, O006-R003 - Version-control-native
- **O006-RSK002 - Non-self-describing corpus**: O006-R002 - Open-format conformance

#### O007 - Complete coverage

*Family: deliverable-quality.*
**True metric:** minimize the likelihood that knowledge within the corpus's
intended scope is absent from it.
**Proxy:** proportion of charter-declared scope areas with at least adequate
concept coverage, plus the count of recoverable held-aside items awaiting
review. *Predicated on a declared charter — without one, "intended scope" has
no referent (see O008).*

**Risks**

- **O007-RSK001** - Lost before intake: Raw material is discarded or forgotten
  before it is ever triaged, so knowledge the team relies on never enters the
  body of knowledge at all.
- **O007-RSK002** - Silent gaps: No signal reveals that an area within the
  intended scope was never captured, so the gap persists unnoticed.
- **O007-RSK003** - Lost at the significance bar: Material relied upon is
  rejected as insubstantial and destroyed, so an intake false-negative becomes
  an invisible, unrecoverable coverage gap.

**Requirements**

- **O007-R001** - Producer-agnostic intake: The product must provide a single
  intake point where any producer can submit raw material the moment it
  arrives, so nothing relied upon is lost before it can be triaged.
- **O007-R002** - Coverage review: The product must periodically surface areas
  within the intended scope that are thin or absent, acting within the
  delegation envelope and escalating for the steward's judgment of what the
  team relies on.
- **O007-R003** - Recoverable rejection: The product must keep material
  rejected at the significance bar in a recoverable held-aside state, so an
  intake false-negative is auditable and reversible rather than a silent loss.

**Risk-Requirement Map**

- **O007-RSK001 - Lost before intake**: O007-R001 - Producer-agnostic intake
- **O007-RSK002 - Silent gaps**: O007-R002 - Coverage review
- **O007-RSK003 - Lost at the significance bar**: O007-R003 - Recoverable rejection, O003-R004 - Significance bar

#### O008 - On-scope focus

*Family: deliverable-quality.*
**True metric:** minimize the proportion of the body of knowledge that falls
outside its intended scope.
**Proxy:** proportion of concepts flagged as scope-divergent against the
charter and not yet resolved. *Predicated on a declared charter.*

**Risks**

- **O008-RSK001** - Input-driven drift: The body of knowledge expands to
  whatever material arrives rather than what its purpose calls for, so
  off-purpose content takes up residence and blurs what it is for.
- **O008-RSK002** - Implicit boundary: The intended scope lives only in the
  steward's head rather than being stated, so no one can adjudicate what belongs
  and drift goes unnoticed.

**Requirements**

- **O008-R001** - Charter-anchored scope: The product must evaluate scope
  against the corpus's **charter** — the required, foundational declaration of
  purpose and scope (see Foundational concepts). Because the charter is
  required, on-scope focus and coverage (O007) always have a referent.
- **O008-R002** - Scope-aware triage: The product must evaluate incoming
  material against the declared scope, admitting or holding it within the
  delegation envelope and escalating out-of-scope ambiguity for the steward
  rather than admitting it automatically.
- **O008-R003** - Scope reconciliation: The product must periodically surface
  content that has drifted outside the declared scope, within the delegation
  envelope, escalating the borderline cases.

**Risk-Requirement Map**

- **O008-RSK001 - Input-driven drift**: O008-R002 - Scope-aware triage, O008-R003 - Scope reconciliation
- **O008-RSK002 - Implicit boundary**: O008-R001 - Charter-anchored scope

#### O009 - Faithful representation

*Family: deliverable-quality.*
**True metric:** maximize the proportion of claims a consumer can rely on as
faithful to what their cited source supports.
**Proxy:** proportion of claims passing the author-time grounding check at or
above threshold, carried as a per-claim grounding signal. (Fidelity checking is
heuristic — hence *maximize the proportion*, not a guarantee: the check is a
proxy for actual faithfulness, and naming the two apart makes the gap a
property of the design, not a hedge.)

**Risks**

- **O009-RSK001** - Distortion in distillation: When raw material is refined
  into a concept, the authored claim overstates, generalizes beyond, or
  otherwise drifts from what the source actually supports, so a consumer relies
  on a claim its own cited source would not back.
- **O009-RSK002** - Indistinguishable grounding: A claim faithful to its source
  and one that outruns it read identically in the body of knowledge, so a
  consumer cannot tell which claims the source actually backs.
- **O009-RSK003** - Invisible review provenance: A human-vetted claim and one
  Wiki integrated autonomously read identically, so a consumer cannot calibrate
  how much to trust the vetting behind a claim.

**Requirements**

- **O009-R001** - Source-grounded authoring: The product must check each
  authored claim against the content of its cited source and, within the
  delegation envelope, admit or hold it — escalating claims the source does not
  support for the steward rather than admitting them silently.
- **O009-R002** - Legible grounding and review tier: The product must surface,
  with each claim, machine-readable signals of (a) how well it is grounded in
  its cited source and (b) who vetted it — human-reviewed versus autonomously
  integrated — so a consumer can tell a source-backed, vetted claim from one
  that needs checking.
- **O009-R003** - Distillation review: The product must present each claim it
  escalates alongside the span of the source it was drawn from, so the steward
  can confirm the claim represents the source before it is integrated.

**Risk-Requirement Map**

- **O009-RSK001 - Distortion in distillation**: O009-R001 - Source-grounded authoring, O009-R003 - Distillation review
- **O009-RSK002 - Indistinguishable grounding**: O009-R002 - Legible grounding and review tier
- **O009-RSK003 - Invisible review provenance**: O009-R002 - Legible grounding and review tier

**Product Decision Records**

- [PDR001 - Claim fidelity as a first-class outcome](drs/PDR001-claim-fidelity-outcome.md)
- [PDR002 - Consumer job as a downstream anchor](drs/PDR002-consumer-downstream-anchor.md)
- [PDR003 - Manage by exception via a delegation envelope](drs/PDR003-manage-by-exception.md)
- [PDR004 - Charter required and foundational](drs/PDR004-charter-required-and-foundational.md)

## See Also

- Product Decision Records: [PDR001](drs/PDR001-claim-fidelity-outcome.md),
  [PDR002](drs/PDR002-consumer-downstream-anchor.md),
  [PDR003](drs/PDR003-manage-by-exception.md),
  [PDR004](drs/PDR004-charter-required-and-foundational.md)
