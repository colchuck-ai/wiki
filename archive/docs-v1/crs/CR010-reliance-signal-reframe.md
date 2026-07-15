# CR010 - Reliance Signal Reframe

Reworded **O002-R003** (coverage review) and **O006-R002** (retirement review) to rest on measurable coverage and retirement signals plus the steward's judgment, rather than a system-measured signal of "what the team relies on," and aligned C008 and C009 to that basis. Scoped to the O002 and O006 outcomes (product) and C008 - Lifecycle & Retirement and C009 - Coverage Review (engineering). Captured by [PDR003 - Reliance Is Steward-Supplied](../product/drs/PDR003-reliance-steward-supplied.md).

## Change

- **O002-R003 (Coverage review)**: reworded from reviewing "which areas of knowledge are represented against what the team actually relies on" to reviewing represented knowledge against the reference's declared scope and other measurable coverage signals (dangling links, agent-proposed gaps), surfacing gaps for the steward, who supplies the judgment of what the team relies on.
- **O006-R002 (Retirement review)**: reworded from "identifying concepts no longer relied upon" to identifying pruning candidates from measurable signals — staleness, source drift, and supersession — and the declared scope, for the steward's retirement judgment.
- **C009 - Coverage Review**: already assembled expected coverage from proxies (dangling links, scaffold scope, agent-proposed gaps); wording aligned so the "relied upon" framing no longer implies a system-measured signal.
- **C008 - Lifecycle & Retirement**: retirement review specified on the same measurable-signal basis.

Before, both requirements invoked "what the team relies on," but nothing in the architecture produces a usage or reliance signal — Query logs nothing and the corpus carries no telemetry. After, each requirement names the computable inputs its review runs on, with reliance supplied as steward judgment. No outcome, risk, or Risk-Requirement Map text changes.

## Rationale

The two requirements promised a capability the architecture never produces, so they were unbuildable as written and a reviewer could not tell when they were met. Reframing to computable signals plus steward judgment matches what any component can compute and gives C008 retirement a concrete basis consistent with C009's existing proxy approach. See PDR003 for the decision and the deferred query-telemetry alternative.

## Affects

- Product: O002-R003 and O006-R002 wording (risks, maps, and outcome text unchanged).
- Engineering: C009 - Coverage Review (wording alignment) and C008 - Lifecycle & Retirement (retirement approach). Captured by [PDR003 - Reliance Is Steward-Supplied](../product/drs/PDR003-reliance-steward-supplied.md).
