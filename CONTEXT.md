# Context — Wiki glossary

The shared language of the project. A glossary and nothing else — no
implementation details, no decisions (those live in PDRs/ADRs), no plans.
When a term below conflicts with how a doc uses a word, the doc is wrong.

## Actors and products

- **Wiki** — the *tool*. Hired by a steward to curate. Wiki is not the
  knowledge; it is the thing that produces and maintains the knowledge.
- **Steward** — the person (a role, possibly held by more than one person over
  time) who hires Wiki and supplies curation *judgment*: declares the charter,
  sets the delegation envelope, and adjudicates escalated exceptions. The
  steward is Wiki's user.
- **Body of knowledge** (also **the corpus**, **the deliverable**) — what Wiki
  produces: a curated set of concepts. It is a *separate product* from Wiki,
  with its own hirer. Wiki is measured on the quality of this deliverable.
- **Consumer** — a **teammate or agent** who hires the *body of knowledge*
  (never Wiki) to answer a question. The consumer's job is the downstream
  anchor from which the deliverable-quality outcomes derive; it is not itself
  measured on Wiki.
- **Producer** — anyone who submits raw material into intake. A producer is not
  necessarily the steward.

## The deliverable's form

- **Concept** — the unit of curated knowledge in the corpus: a distilled,
  cited claim (or small cluster of claims) about one thing.
- **Claim** — an assertion within a concept, carrying a citation to the source
  it was drawn from.
- **Structured substrate** — the corpus is one set of plain-text, open-format
  files that serves **two access modes**: a *human view* (rendered index,
  cross-links, badges) and an *agent-parseable form* (the same facts as
  machine-readable fields). Anything legible to a human is machine-readable
  *first*, presentation second.
- **Citation** — a concept's reference to its source. For a captured-snapshot
  source it carries **two** structured references: the **durable snapshot**
  (the frozen form the claim was drawn from — the resolution target for
  traceability and fidelity) and the **live-origin locator** (the external URL,
  read only to detect drift). Neither lives in prose.
- **OKF (Open Knowledge Format)** — the external, published, versioned open
  standard the corpus conforms to. Wiki *consumes* OKF; defining it is a
  non-goal. Portability has two layers: *substrate* (plain text + version
  control — unconditional) and *semantic* (OKF conformance — an open,
  implementation-independent spec).

## Governance

- **Policy anchor** — the shared pattern for a steward-declared, persistent,
  in-corpus, revisable-in-place statement that governs Wiki's behavior. The
  charter and the delegation envelope are both policy anchors; they differ in
  that the charter is *required* and the envelope is *optional*.
- **Charter** — the **required**, foundational policy anchor declaring the
  corpus's purpose and scope. Provisional at first and sharpened over time. It
  is *load-bearing*: triage, the significance bar, duplication, and coverage
  all read against it. A consumer can read it to know what the corpus is for.
- **Delegation envelope** — the optional policy anchor by which the steward
  declares what Wiki may do **autonomously** versus what it must **escalate**.
  Tuned as the steward's trust in Wiki grows.
- **Manage by exception** — Wiki acts autonomously within the envelope and
  escalates only genuinely ambiguous cases; the steward is an *auditor* of the
  common path, not a synchronous gate on it.
- **Decision-provenance** — the per-item record of *how* an item was handled:
  the outcome (integrated / held aside / retired), the actor (Wiki-auto vs.
  steward), the envelope version, and the time. Makes autonomous action
  auditable.
- **Review tier** — the consumer-visible signal of *who vetted* a claim
  (human-reviewed vs. autonomously integrated). Orthogonal to the *grounding
  signal*, which says how well a claim matches its source.
- **Hold aside** — the recoverable, not-integrated state for material below the
  significance bar. Intake rejection is **non-destructive**: below-bar material
  is held, not discarded, so a wrong call is auditable and reversible.

## Cross-cutting principles

- **Transferable stewardship** — the corpus should hold everything a steward
  needs to steward it, so curation knowledge does not live in one head. The
  steward-side mirror of the product's core promise. Minimize what a steward
  must hold that is not recoverable from the corpus.
- **Deprecation over deletion** — knowledge and material are marked and kept
  (deprecated, superseded, held aside), not destroyed; genuine purging is an
  explicit, separate act.
- **Proxy metric** — a corpus property Wiki can observe that stands in for a
  consumer-experienced outcome Wiki cannot directly see. Each
  deliverable-quality outcome states both its consumer-value *true metric* and
  its observable *proxy*.
