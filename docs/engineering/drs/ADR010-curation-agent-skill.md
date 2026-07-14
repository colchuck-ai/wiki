# ADR010 - Curation Agent Skill and Operations

Realize Wiki's delivery form as a **curation Agent Skill** exposing four
intent-shaped operations — **Ingest**, **Query**, **Survey**, and **Lint** —
through which the steward drives the pipeline and standing capabilities by intent,
and adopt the **OKF Knowledge Bundle** (SPEC §2/§3) as the concrete meaning of
"bundle" throughout Wiki.

## Context

The Technology Choices commit to agent-driven authoring "realized as an Agent
Skill," but no operation surface was ever defined: there was no statement of how
a steward actually invokes triage, integration, revalidation, or reconciliation.
This is the delivery-form gap called out in the implementation-gaps assessment —
the pipeline and standing capabilities existed as components with no invocation
home.

Separately, C010 - Scaffold and [ADR005 - Guided Fork-and-Migrate](ADR005-guided-fork-and-migrate.md)
fork "a new bundle," but "bundle" was never defined in-repo. Vendoring OKF
resolved this: OKF §2 defines a **Knowledge Bundle** as a self-contained,
hierarchical collection of knowledge documents — the unit of distribution — and
§3 says it ships as a git repository (recommended), a tarball, or a subdirectory.
The delivery decision should record that "bundle" is now a concretely defined
term, not an open question.

## Options

- **Leave the delivery form undefined**: keep "an Agent Skill" with no
  operations. Rejected — it is the standing blocker in the gaps assessment; the
  steward has no defined way to act.
- **One operation per component**: expose triage, authoring, indexing,
  revalidation, reconciliation, retirement each as its own command. Rejected —
  leaks the architecture into the steward's surface and is far more granular than
  the way a steward actually works (by intent, not by component).
- **Three intent-shaped operations — Ingest / Query / Lint**: fold every periodic
  review, including scaffold-driven coverage analysis, into a single Lint pass.
  Rejected — it conflates two different steward intents. Lint's other checks are
  *corrective and inward-facing*: they find defects in what already exists and
  their output is edits to the corpus. Scaffold-driven coverage analysis is
  *generative and outward-facing*: it finds what is **absent** against the
  reference's declared purpose, and its output is a **sourcing agenda** — a list
  of thin or empty areas the steward acts on by going to *find sources*, not by
  editing the corpus. Burying that inside a hygiene pass both mismatches the
  intent and hides the scaffold's signature payoff.
- **Four intent-shaped operations — Ingest / Query / Survey / Lint** *(chosen)*: a
  small surface partitioned by distinct steward intent — add material, ask the
  reference, assess it against what it is *for*, and keep it well-formed. The
  fourth verb is justified by intent, not by component granularity (the criterion
  that rejected one-verb-per-component), so it extends the surface on the same
  principle rather than against it.

## Decision

Adopt the **curation Agent Skill** with four operations:

- **Ingest** — capture (C001 - Inbox) → scope triage (C002 - Scope Triage) →
  integration authoring (C003 - Integration Authoring). The steward supplies raw
  input and a keep / merge / deprecate / discard verdict; for a merge, the
  steward also chooses the supersede / fold-in / correct reconciliation
  (per [ADR004 - Merge Reconciliation Steering](ADR004-merge-reconciliation-steering.md)).
- **Query** — the steward asks the reference a question; the Skill navigates via
  C005 - Index & Navigation and answers with citations. A valuable synthesized
  answer may be filed back into the Inbox as a producer deposit (see
  [CR006 - Query feedback loop](../../crs/CR006-query-feedback-loop.md)), so
  exploration compounds. Wiki owns no separate query surface — query is a
  convenience over the curated corpus, consistent with the product's query
  non-goal.
- **Survey** — the scaffold-relative assessment: measure what the corpus holds
  against what the scaffold says it is *for*. Two directions of one question:
  **coverage gaps** — areas the reference is expected to cover but that are thin
  or absent (C009 - Coverage Review, drawing on dangling links as not-yet-written
  knowledge, the declared scaffold scope, and agent-proposed areas) — and **scope
  drift** — content that has drifted outside the declared purpose
  (C010 - Scaffold scope reconciliation). Survey's coverage output is a
  **sourcing agenda**: a prioritized list of what is missing that the steward acts
  on by going to find sources, which then re-enter through Ingest — so Survey is
  the front of the sourcing loop, not a corpus edit. It requires a declared
  scaffold to be meaningful; with none, only the dangling-link and agent-proposed
  signals apply.
- **Lint** — the scaffold-independent hygiene pass: find and surface defects in
  what already exists. Conformance validation (C004 - OKF Conformance),
  referential integrity — dangling/broken references surfaced for repair
  (C005 - Index & Navigation), source revalidation (C006 - Source Revalidation),
  recency and staleness review (C007 - Currency Tracking), and retirement review
  (C008 - Lifecycle & Retirement). Lint's output is corrective edits to the
  corpus, not a sourcing agenda. A dangling link feeds both operations by
  different lenses: Lint sees a broken reference to repair; Survey sees a
  not-yet-written concept to source.

The fuzzy judgment classifiers the framework already acknowledges — scope
adjudication (C002) and core-vs-periphery detection (C010) — are realized as
Agent-Skill prompt criteria anchored on the declared scaffold and co-evolved with
the steward, not as fixed algorithms.

"Bundle" is the **OKF Knowledge Bundle** (SPEC §2/§3): a git repository
(recommended), tarball, or subdirectory. Fork-and-migrate (ADR005) forks a new
OKF bundle.

## Consequences

- Enabling: gives Wiki a concrete, intent-shaped interaction surface and closes
  the delivery-form gap; the standing capabilities gain invocation homes — hygiene
  checks in Lint, scaffold-relative assessment in Survey.
- Enabling: Survey first-classes the scaffold's signature payoff — turning the
  declared purpose into an active coverage analysis whose output is a sourcing
  agenda — so the differentiator is visible in the surface instead of buried as
  one bullet in a hygiene pass, and gap-driven sourcing becomes a named loop that
  feeds Ingest.
- Enabling: "bundle" is concretely defined by the vendored spec, so C010/ADR005
  fork-and-migrate has a definite unit to fork.
- Cost: the Agent Skill is the single delivery vehicle, so Wiki carries a
  Skill-runtime dependency.
- Cost: classifier quality rests on prompt criteria plus the scaffold rather than
  deterministic rules — consistent with ADR004 and ADR005 already acknowledging
  these as fuzzy steward-adjudicated judgments.
- Cost: the operations must stay within the low-volume, single-steward envelope
  ([ADR009 - Low-Volume Operating Envelope](ADR009-low-volume-operating-envelope.md));
  they are not designed for machine-volume unattended throughput.
- Cost: a fourth verb enlarges the surface and introduces a boundary the steward
  must learn — "is the corpus well-formed and current?" (Lint) versus "does it
  match what it is for?" (Survey). Dangling links deliberately appear in both,
  which must be explained so the overlap reads as two lenses, not a contradiction.
