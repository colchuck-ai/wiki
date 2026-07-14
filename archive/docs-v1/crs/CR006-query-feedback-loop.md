# CR006 - Query Feedback Loop

Made explicit, on the product README and C001 - Inbox, that knowledge synthesized while *consuming* the reference re-enters curation through the producer-agnostic Inbox — closing the compounding loop without giving Wiki a query surface. A documentation clarification of an existing capability; no requirement, outcome, or decision changes.

## Change

Before, the product README declared query and personal synthesis non-goals and left the fate of a valuable answer produced while consuming the reference unstated. The producer-agnostic Inbox (C001) already accepts material from any producer, but nothing named the case where the producer is a *consumer* of the reference — a teammate, a consuming agent, or the companion Zettelkasten — depositing a synthesized answer back for adjudication. The compounding-knowledge loop that the whole curation discipline exists to serve was therefore left implicit.

After:

- **Product README** — the producer-agnostic intake narrative and the curate→synthesize composition note state that consumption feeds back into curation: an answer or synthesis produced downstream can be deposited into the Inbox as any other producer's material would be, then triaged and integrated on its own merits. The Non-goals wording is sharpened so "Wiki owns no query surface" and "consumption re-enters as a producer deposit" read as complementary, not contradictory — the non-goal is preserved, not weakened.
- **C001 - Inbox** — a consuming agent (and the companion Zettelkasten) is named as an example producer, alongside humans and automated producers, so the producer-agnostic guarantee visibly covers the feedback path.

## Rationale

The Inbox is already the single, producer-agnostic entry point with one adjudication gate. A consumption-derived answer is just another producer's deposit, so it re-enters through the existing intake-and-triage path — no query capability, no second entry point, and no violation of the query non-goal. Naming this makes the compounding loop (the reason curation is worth the effort) a stated property of the design rather than an accident a reader has to infer. Because the mechanism already exists, this is a clarification, not a new decision — no ADR or PDR is warranted.

## Affects

- Product: the product README's producer-agnostic intake narrative, the curate→synthesize composition note, and the Non-goals wording. No outcome or requirement text changes.
- Engineering: C001 - Inbox (a consuming agent named as an example producer). No behavior change — the producer-agnostic intake already admits this path.
