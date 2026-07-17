# PDR002 - Question-based retrieval as a first-class discovery path

We decided to model retrieval-by-question as its own risk (**O005-RSK003 - Vocabulary mismatch**) and requirement (**O005-R004 - Question-based retrieval**) under **O005 - Fast discovery**, rather than folding it into the navigable index, leaning on the agent-skill delivery's inherent reading, or treating it as out of scope for the low-volume envelope.

This decision affects **O005 - Fast discovery** (`docs/product/README.md`): it adds one risk and one solution-free requirement and does not change O005's metric. It leaves an open follow-up — O005-R004 has no engineering owner yet (see Consequences).

## Context

O005 - Fast discovery measures the time to locate the knowledge relevant to a **question**. Its risks and requirements, however, covered only two ways of finding: browsing a listing of what exists (O005-R001, against flat sprawl) and traversing stated relationships (O005-R002, with O005-R003 keeping those links from dead-ending). Both assume the consumer can recognize the right entry from how the corpus is arranged.

The characteristic discovery failure is not sprawl or a dead link — it is that the consumer asks in different words than the ones the relevant knowledge was titled and filed under. A listing and a cross-link graph do not help when you cannot guess the heading; the knowledge is present but unreachable by scanning. The consumer then falls back to asking the one person who holds it — the exact routing the product spine promises to eliminate ("find and rely on what we know without routing every question through me"). This is the consumer-facing half of that promise, and it had no risk and no requirement.

## Options

- **Fold it into O005-R001 - Navigable index.** An index is browse-by-structure: it still requires the consumer to recognize the right heading. Vocabulary mismatch defeats a listing as surely as it defeats a folder tree, so the gap would stay hidden inside a requirement that already reads as satisfied.
- **Treat question-answering as inherent to the agent-skill delivery** — the curation agent reads the corpus and answers — so no product requirement is needed. But outcomes and requirements are stated independent of the solution, and leaving retrieval implicit is exactly why the gap went unmodeled. An unstated capability is untraceable and unverifiable: nothing downstream is obliged to satisfy it, and no check can tell whether it works.
- **Treat question retrieval as out of scope for the low-volume, single-steward envelope.** The envelope bounds submission volume and the steward's review effort, not the consumer's need to find things. O005 explicitly frames discovery around a question, and the spine promises finding without routing through the steward; dropping question retrieval would contradict both.
- **Add it as its own risk and requirement (chosen).** Gives the signature discovery failure a dedicated, measurable risk and a verifiable requirement, kept solution-free so the mechanism is an engineering decision, not a product commitment.

## Decision

Add **O005-RSK003 - Vocabulary mismatch** and **O005-R004 - Question-based retrieval** under O005. Keep O005-R004 solution-free — it states that a consumer must be able to retrieve the concepts relevant to a question posed in their own terms, not any particular mechanism (keyword search, semantic match, or the agent reasoning over the corpus). O005 remains *minimize the time to locate*, not a guarantee of a hit.

Retrieval-by-question is genuinely distinct from browsing and traversal: a consumer can face a complete index and a well-linked graph and still be unable to find present knowledge whose vocabulary they do not share. Keeping it a separate risk and requirement lets that failure be measured and mitigated on its own terms.

## Consequences

- Discovery is now decomposed across all three modes a consumer actually uses: browse (O005-R001), traverse (O005-R002, with O005-R003 protecting traversal from dead ends), and query (O005-R004). No silent gap remains between "the knowledge exists" and "a consumer can find it."
- **O005-R004 currently maps to no component.** Like O009 at its birth ([PDR001](PDR001-claim-fidelity-outcome.md)), the requirement is stated but not yet traced. A follow-up engineering trace must decide where question-based retrieval lives — an extension of C005 - Index & Navigation, a dedicated retrieval component, or a capability of the Query operation / curation Agent Skill — and add the row to the Requirement-Component Map. Because O005-R004 is solution-free, that trace is free to choose the mechanism (the agent reasoning over the corpus vs. a built search/index) without reopening this product decision.
- Retrieval quality is heuristic — mapping a question to the relevant concepts is a judgment that can miss — so the outcome stays *minimize the time to locate*, consistent with the fidelity outcome's heuristic framing (PDR001).
