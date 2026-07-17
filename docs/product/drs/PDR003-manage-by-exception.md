# PDR003 - Manage by exception via a delegation envelope

We decided that Wiki should **act autonomously within a steward-declared
delegation envelope and escalate only exceptions**, rather than gate every
admission and upkeep action on synchronous steward review — and that this
autonomy is made trustworthy by pairing it with **decision-provenance** and a
consumer-visible **review tier**.

This decision affects the product statement (`docs/product/README.md`): it adds
the delegation envelope and decision-provenance/review-tier as foundational
concepts, adds "manage by exception" as an operating principle, adds O004-R004
(delegated autonomy) and O004-RSK004 (review-gate toil), and rewrites the verb
of the admission/upkeep requirements (O003-R002/R003/R004, O007-R002,
O008-R002/R003, O009-R001/R003) from "flag for the steward" to "act within the
envelope, escalate exceptions."

## Context

The job's headline promise is to let consumers self-serve *"without routing
every question through the one person who holds it in their head."* The
deliverable-quality outcomes dissolve that **consumer** bottleneck. But the
write path terminated almost everywhere in a synchronous **steward** gate —
distillation review before integration, scope triage, dedup, significance,
retirement, coverage. Since raw material "accumulates faster than anyone can
absorb it," a per-item human gate makes the steward's effort scale with input
volume: the reader's bottleneck relocated to the writer, and precisely the toil
O004 exists to prevent, moved from the *hands* (formatting) to the *eyes*
(judging).

Removing the gate, though, means the corpus will hold claims no human reviewed.
For a distiller whose value is "trust on sight," that is only safe if autonomous
action is legible and reversible.

## Options

- **Steward gates everything** (status quo): maximal trust per item, but the
  corpus cannot grow faster than one person can review, contradicting the
  premise.
- **Per-outcome gating** (gate O008/O009, delegate the rest): partial relief,
  but still a synchronous gate on the highest-volume paths and no general model
  for autonomy.
- **Delegated envelope with exception-only escalation** (chosen): the steward
  sets policy and thresholds once; Wiki acts within them and escalates only
  ambiguous cases; the steward becomes an *auditor* of the common path.

## Decision

Adopt **manage by exception**. The steward declares a **delegation envelope**
(an optional policy anchor — the same declared/persistent/in-corpus/revisable
pattern as the charter, tuned as trust grows) governing what Wiki does
autonomously versus escalates. Wiki acts within it and escalates exceptions.

Pair autonomy with two safeguards, without which it is not trustworthy:

- **Decision-provenance** — every item records how it was handled: outcome
  (integrated / held aside / retired), actor (Wiki-auto vs. steward), envelope
  version, and time. Recorded into the change history (O002-R002).
- **Review tier** — a consumer-visible signal of *who vetted* a claim
  (human-reviewed vs. autonomously integrated), orthogonal to the grounding
  signal (O009-R002).

Autonomy is trustworthy because it is legible and reversible, not because the
machine is trusted blindly.

## Consequences

- The admission/upkeep requirements change verb: "act within the envelope,
  escalate exceptions." O004 gains R004 (delegated autonomy) against new
  RSK004 (review-gate toil).
- O009-R002 gains the review-tier axis; O009 gains RSK003 (invisible review
  provenance). O002-R002 change history now carries decision-provenance.
- Destructive autonomous actions are constrained by *deprecation over deletion*:
  in particular the significance bar must be non-destructive (see the
  significance-bar treatment under O003-R004 / O007-R003), so an autonomous
  reject is auditable and reversible rather than a silent loss.
- The charter and envelope are two distinct policy anchors sharing one pattern;
  their relationship is recorded in
  [PDR004](PDR004-charter-required-and-foundational.md).
