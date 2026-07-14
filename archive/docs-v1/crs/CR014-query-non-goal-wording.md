# CR014 - Query Non-Goal Wording

Sharpened the product README's query non-goal so the curation Agent Skill's **Query** operation and the "no query surface" non-goal read as complementary rather than contradictory. A documentation clarification of an existing position; no requirement, outcome, or decision changes. Scoped to the product README.

## Change

- **Product README (Non-goals)**: the non-goal "Nor does Wiki own a query surface" is sharpened to name what is excluded — a persisted, standalone query product — and what Query is — a stateless read convenience over the curated corpus — so it no longer appears to contradict the Query operation ADR010 ships.

Before, the flat statement "Wiki does not own a query surface" sat in apparent tension with the Query operation, and a reader had to infer the reconciliation. After, the non-goal states the boundary precisely (no persisted, standalone query product; Query is a stateless read convenience), preserving the non-goal without contradicting the operation.

## Rationale

Aligns the product non-goal with the delivery surface (ADR010), continuing the reconciliation [CR006](CR006-query-feedback-loop.md) began. Because Query already exists as a convenience over the corpus and no capability changes, this is a one-line wording clarification — no ADR or PDR is warranted.

## Affects

- Product: the README Non-goals wording. No outcome, requirement, or decision text changes.
