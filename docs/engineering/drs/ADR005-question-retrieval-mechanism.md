# ADR005 — Question-retrieval mechanism owner

## Status
Accepted (2026-07-17). Engineering clean-room rewrite, Chunk A. Realizes PDR005.

## Context
O005-R004 (question-based retrieval) requires that a consumer retrieve concepts relevant
to a question posed *in their own terms* — not only browse the index or follow
cross-links — because the signature discovery failure is vocabulary mismatch: the
knowledge is present but the consumer cannot guess the heading (O005-RSK004). PDR005
keeps the requirement **solution-free** and explicitly hands the mechanism to
engineering: "an agent reasoning over the corpus vs. a built search/index." The product
also frames retrieval as **heuristic** — *minimize the time to locate*, not a guarantee
of a hit. This is a genuinely new capability with no prior decision in the old tree.

## Options
- **Fold into the index (C009):** but an index is browse-by-structure — it still requires
  recognizing the right heading, so it does not address vocabulary mismatch, and folding
  it hides the gap inside a component that reads as already satisfied.
- **Assume the delivery agent just does it:** an unstated capability is untraceable and
  unverifiable (PDR005 rejected this at the product level).
- **Its own component, mechanism deliberately open** (chosen).

## Decision
[C010 Question Retrieval](../components/C010-question-retrieval.md) is a component
distinct from [C009 Discovery & Index](../components/C009-discovery-and-index.md),
exposing a single heuristic surface: `retrieve(question) -> [RankedConcept]`. It is
separate from C009 precisely because C009 is *deterministic navigational structure* while
C010 is *heuristic retrieval* — different guarantees, different test strategies.

The **mechanism is left open behind the interface**: an agent reasoning over the corpus,
or a built search/embedding index, or a hybrid. Nothing else in the architecture depends
on which — the interface hides it — so the choice can be made, measured, and swapped in
the C010 Phase-3 spec without reopening this decision or touching any other component.

## Consequences
- Discovery is complete across all three modes a consumer uses — browse (C009), traverse
  (C009 graph), query (C010) — with no silent gap between "the knowledge exists" and "a
  consumer can find it."
- Because the mechanism is behind a small interface, C010 is a clean seam: one adapter
  today, a different one tomorrow, with no ripple.
- Retrieval quality stays honestly heuristic (may miss); success is measured as *time to
  locate*, consistent with the heuristic framing of O009.
- **Reversal:** the *mechanism* is cheap to change (that is the point); the *decision to
  own retrieval as a distinct component* is the load-bearing, recorded part.
