# PDR005 - Question-based retrieval as a discovery path

We decided to model retrieval-by-question as its own risk (**O005-RSK004 -
Vocabulary mismatch**) and requirement (**O005-R004 - Question-based
retrieval**) under **O005 - Fast discovery**, rather than folding it into the
navigable index, leaning on the delivery agent's inherent reading, or treating
it as out of scope.

This decision affects **O005 - Fast discovery** (`docs/product/README.md`): it
adds one risk and one solution-free requirement and does not change O005's true
metric. (Reproduced from earlier work that was removed in a merge collision.)

## Context

O005 measures the time to locate the knowledge relevant to a **question**. Its
risks and requirements otherwise cover only two ways of finding: browsing a
listing (O005-R001) and traversing stated relationships (O005-R002, with
O005-R003 keeping links from dead-ending). Both assume the consumer can
recognize the right entry from how the corpus is arranged.

The characteristic discovery failure is neither sprawl nor a dead link — it is
that the consumer asks in different words than the ones the knowledge was titled
and filed under. A listing and a cross-link graph do not help when you cannot
guess the heading; the knowledge is present but unreachable by scanning. The
consumer then falls back to asking the one person who holds it — the exact
routing the spine promises to eliminate. This is the consumer-facing half of
that promise, and without this requirement it had no risk and no requirement.

## Options

- **Fold it into O005-R001 - Navigable index.** An index is browse-by-structure;
  it still requires recognizing the right heading. Vocabulary mismatch defeats a
  listing as surely as a folder tree, so the gap would hide inside a requirement
  that already reads as satisfied.
- **Treat question-answering as inherent to the delivery agent** — so no product
  requirement is needed. But requirements are stated independent of the
  solution; an unstated capability is untraceable and unverifiable.
- **Treat it as out of scope.** O005 explicitly frames discovery around a
  question, and the spine promises finding without routing through the steward;
  dropping it would contradict both.
- **Add it as its own risk and requirement (chosen).** Gives the signature
  discovery failure a dedicated, measurable risk and a verifiable requirement,
  kept solution-free so the mechanism is an engineering decision.

## Decision

Add **O005-RSK004 - Vocabulary mismatch** and **O005-R004 - Question-based
retrieval** under O005. Keep O005-R004 solution-free: a consumer must be able to
retrieve the concepts relevant to a question posed in their own terms — not any
particular mechanism (keyword search, semantic match, or the agent reasoning
over the corpus). O005 remains *minimize the time to locate*, not a guarantee of
a hit.

Retrieval-by-question is genuinely distinct from browsing and traversal: a
consumer can face a complete index and a well-linked graph and still be unable
to find present knowledge whose vocabulary they do not share. Keeping it a
separate risk and requirement lets that failure be measured and mitigated on its
own terms.

## Consequences

- Discovery is now decomposed across all three modes a consumer uses: browse
  (O005-R001), traverse (O005-R002, protected by O005-R003), and query
  (O005-R004). No silent gap remains between "the knowledge exists" and "a
  consumer can find it."
- **O005-R004 needs a component owner in the engineering rewrite.** Because it is
  solution-free, the engineering clean-room is free to choose the mechanism (an
  agent reasoning over the corpus vs. a built search/index) without reopening
  this product decision. Captured in `docs/engineering/REWRITE-PLAN.md` as a
  trace target.
- Retrieval quality is heuristic — mapping a question to the relevant concepts
  can miss — so the outcome stays *minimize the time to locate*, consistent with
  the heuristic framing of O009 ([PDR001](PDR001-claim-fidelity-outcome.md)).
