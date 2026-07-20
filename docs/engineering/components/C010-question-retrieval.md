# C010 — Question Retrieval

**Role.** Heuristic free-form lookup: a question in the consumer's own words →
the relevant concepts, ranked — for the consumer who cannot guess the heading
(vocabulary mismatch, O005-RSK004).

## Capabilities

The whole component is one call — see [INTERFACES.md](../INTERFACES.md) § C010:

- `retrieve(question) -> [RankedConcept]` — map a question posed in the
  consumer's own terms to the concepts that most likely answer it, best-first.

It is **heuristic, not a lookup**: mapping words a consumer chooses to concepts
someone else titled and filed is an inference, so the result **may miss**, may
rank a weak match above a strong one, and may return nothing. The ranking is the
honest shape of that uncertainty — an ordered set of candidates for the consumer
to judge, not an authoritative answer. This is deliberate and matches the O005
framing: the goal is to *cut the time to locate*, not to guarantee a hit.

`RankedConcept` is a C010-local type (a `ConceptId` plus a relevance ordering —
[DECOMPOSITION.md](../DECOMPOSITION.md#shared-types)); it is not shared vocabulary
because nothing else consumes it. What "relevance" means is internal to the
chosen mechanism and stays behind the interface.

## Data model

**Nothing persistent** under the chosen initial mechanism (below). C010 reads the
corpus — the single source of truth C008 writes — at query time and owns no
store of its own. There is no index to build, stamp, invalidate, or keep in sync;
correctness cannot lag the corpus because there is no derived artifact to lag it.

The interface does not commit to this. Should the mechanism change to a built
search or embedding index, that index is a **derived retrieval structure**
recomputed from the corpus (like C009's index and C005's currency — a function of
the corpus, never a second source of truth), materialized entirely behind
`retrieve`. Either way C010 is **read-only over the corpus** and writes nothing.

## Behavior

**Chosen initial mechanism: an agent reasoning over the corpus at query time.**
`retrieve(question)` hands the question and the corpus to a reasoning agent that
reads concepts and returns the ranked candidates it judges relevant. No index, no
embeddings, no build step.

Rationale:

- **The corpus is small, plain-text, and self-describing.** It is OKF-conformant
  files in version control (O006), machine-readable-first by invariant. An agent
  can read it directly; at current scale, reasoning over the text costs little and
  needs no infrastructure.
- **It attacks vocabulary mismatch head-on.** The failure is that the consumer's
  words differ from the words the knowledge was filed under. A reasoning agent
  matches on *meaning*, not on tokens the consumer would have to already know —
  which is exactly what a keyword index cannot do and the deterministic structure
  of C009 does not attempt.
- **It reuses structure C009 already exposes.** The machine-readable index and
  reason-annotated cross-links are legible context an agent can lean on, so C010
  need not rebuild navigational structure to exploit it.
- **Lowest-commitment starting point.** No persistent artifact means nothing to
  keep current and no way for retrieval to silently drift from the corpus — the
  cheapest mechanism that can be *measured* before investing in a built index.

**Swappable with no ripple.** If measurement shows the agent-over-corpus approach
is too slow, too costly, or too imprecise as the corpus grows, the mechanism can
move to a built search/embedding index or a hybrid **inside this spec**, behind
the unchanged `retrieve` signature. ADR005 established that nothing else depends
on which mechanism is used; this choice honors that and does not narrow it.

**Why C010 is distinct from C009.** C009 is *deterministic navigational
structure* — the index, the cross-link graph, inbound/dangling queries. It answers
"what exists and how is it connected," and it is exact and repeatable. But browse
and traverse both require the consumer to **recognize the right heading** — which
is precisely what a consumer suffering vocabulary mismatch cannot do. C009 cannot
close that gap without ceasing to be deterministic. C010 is the heuristic path
that does: different guarantee, different test strategy (relevance quality, not
structural correctness), different component (ADR005). Folding it into C009 would
hide the gap inside a component that reads as already satisfied.

## Relationships

- **Reads the corpus, writes nothing.** Read-only, like C009 — it produces a view,
  it never mutates concepts.
- **Depends on nothing but the corpus read.** Per the
  [INTERFACES.md dependency direction](../INTERFACES.md#dependency-direction-no-cycles),
  `C010 Question Retrieval → (reads corpus)` — no arrow to any other component.
  It may *read* C009's machine-readable structure as context, but takes no
  dependency on C009's interface: C010 must function on the raw corpus alone.
- **Distinct from C009** (heuristic vs deterministic) and from C008 (which owns
  every write). No component calls C010; it is a leaf consumer-facing surface.

## Decisions

- [ADR005 — Question-retrieval mechanism owner](../drs/ADR005-question-retrieval-mechanism.md):
  C010 is a component **distinct** from C009 because it is heuristic where C009 is
  deterministic — that separation is the load-bearing, recorded decision. The
  **mechanism is left open** behind the interface (agent-over-corpus vs built
  search/embedding index vs hybrid). This spec **chooses agent-over-corpus as the
  initial mechanism** and records the rationale above; the choice is reversible
  and component-internal, changeable without reopening ADR005 or touching any
  other component.

## Success criteria

Tied to the **O005 true metric — minimize the time to locate** the knowledge
relevant to a question — measured against the consumer's downstream job, and
**honestly heuristic**:

- **Success is a reduction in time-to-locate for vocabulary-mismatch questions:**
  a consumer who cannot guess the heading reaches the relevant concept faster via
  `retrieve` than by scanning the index or falling back to asking the person who
  holds the knowledge (O005-RSK004). Ranking helps here — the relevant concept
  surfacing near the top is what saves time.
- **It is NOT measured as a hit guarantee.** Consistent with O005's heuristic
  framing, `retrieve` may miss or return nothing; that is a known, designed
  limitation, not a defect. A miss falls back cleanly to C009 browse/traverse and
  costs no more than not having tried.
- **It writes nothing, so it cannot degrade any other outcome.** Retrieval quality
  is the only thing at stake; provenance, currency, and integrity are untouched
  because C010 is read-only.
- **The mechanism is judged on this metric, not on its internals.** Whether
  agent-over-corpus or a future built index, the bar is the same: shorter
  time-to-locate for questions the consumer cannot phrase in the corpus's own
  words. That is what would justify swapping the mechanism.
