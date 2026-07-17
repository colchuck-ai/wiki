# CR009 - O005 question-based retrieval path

Extended **O005 - Fast discovery** with the retrieval-by-question path it was missing: added risk **O005-RSK003 - Vocabulary mismatch** and requirement **O005-R004 - Question-based retrieval**, and mapped them. The decision and its alternatives are recorded in [PDR002](../product/drs/PDR002-question-based-retrieval.md).

## Change

Before: O005 measured the time to locate knowledge relevant to a **question**, but every risk and requirement covered only browsing a listing (O005-R001, against flat sprawl) or traversing cross-links (O005-R002, with O005-R003 keeping them from dead-ending). The characteristic discovery failure — a consumer asking in different terms than the ones the knowledge was titled and filed under, so present knowledge is unreachable by scanning — had no risk and no requirement. That failure routes the consumer back to the one person who holds the knowledge, contradicting the product spine's promise of finding "without routing every question through me."

After:

- **O005 Risks** gains **O005-RSK003 - Vocabulary mismatch**.
- **O005 Requirements** gains **O005-R004 - Question-based retrieval**, kept solution-free: the product must let a consumer retrieve the concepts relevant to a question posed in their own terms — not only browse the listing or follow cross-links — so present knowledge is findable without the consumer already knowing how it was named or filed.
- **O005 Risk-Requirement Map** gains the row **O005-RSK003 → O005-R004**.
- The product statement's See Also gains PDR002 and this CR.

## Rationale

Browsing and traversal both assume the consumer can recognize the right entry from how the corpus is arranged; neither helps when the consumer cannot guess the heading. Retrieval-by-question is therefore a distinct discovery mode, not a facet of the index, and it earns its own risk and requirement so the failure can be measured and mitigated on its own terms. The requirement is stated solution-free so the retrieval mechanism stays an engineering decision. The full set of alternatives — fold into the index, lean on the agent-skill's inherent reading, or scope it out for the low-volume envelope — is weighed in PDR002.

## Affects

- **O005 - Fast discovery** — gains O005-RSK003, O005-R004, and the map row; its metric is unchanged.
- **Wiki product statement** (`docs/product/README.md`) — See Also gains PDR002 and this CR.
- **PDR002 - Question-based retrieval as a first-class discovery path** — the decision this change record realizes.
- **Engineering tree** — O005-R004 has no component owner yet; a follow-up trace must add its row to the Requirement-Component Map (recorded as a PDR002 consequence).
