# CR007 - Claim grounding trace: O009 requirements to C003

Traced the fidelity outcome **O009 - Faithful representation** to the engineering tree, discharging the follow-up [PDR001](../product/drs/PDR001-claim-fidelity-outcome.md) left open. Mapped O009-R001 (source-grounded authoring), O009-R002 (legible grounding), and O009-R003 (distillation review) to **C003 - Integration Authoring**, added the author-time **grounding** capability to C003, and recorded the per-claim grounding-signal marker as a **C004 - OKF Conformance** convention. The seam placement is decided with alternatives in [ADR015](../engineering/drs/ADR015-claim-grounding-seam.md).

## Change

Before: O009 existed as a product outcome with three requirements but no engineering owner — the Requirement-Component Map had no O009 rows, and PDR001 carried the standing consequence "O009's three requirements currently map to no component; a follow-up engineering trace is required to assign owners." The outcome was stated but unbuildable-as-traced.

After:

- **Requirement-Component Map** gains three rows: **O009-R001 → C003**, **O009-R002 → C003**, **O009-R003 → C003**.
- **C003 - Integration Authoring** claims O009-R001/R002/R003 in its intro and gains a self-contained, author-time grounding capability: `check_grounding(draft_concept) → claim_grounding[]` — an internal, advisory, read-only agent pass (the same shape as `propose_links`) that pairs each authored claim with the cited source span it was drawn from and a support verdict. O009-R001 is its non-`supported` subset, flagged for the steward as part of authoring's recommend-and-confirm; O009-R003 is its output presented as the distillation review (claim beside source span) before the write commits; O009-R002 is its verdict persisted as a per-claim grounding signal, materialized as a terminal verb effect of `create`/`revise` alongside recency, history, and reindex. The grounding check flags and surfaces; it never gates authoring by itself (intent in, work out).
- **C004 - OKF Conformance** records the per-claim grounding-signal marker among the authoring conventions it owns — shape only, a Wiki convention under OKF's producer-extension allowance (§4.1/§4.2/§9), no OKF change. The signal's value is C003's; its format is C004's, the same split C004/C007 already have for `timestamp`.

## Rationale

The three O009 requirements share one computation — a heuristic judgment of whether a cited source supports a claim — and all three anchor at authoring, where the claim has just been composed and its source promoted. Triage (C002) has no authored claim to check; C006 does mechanical byte-drift detection and refuses semantic judgment, a different axis. C003 is the only component that holds the claim, the source, and the citation machinery at the same moment, so the grounding judgment belongs there.

It is added as a *named, self-contained* capability rather than folded into prose so the fidelity judgment has a legible, testable owner — the engineering echo of PDR001's reason for making O009 its own outcome instead of hiding it inside O001. It is *not* given a dedicated component because it has exactly one caller today; a delegation seam is documented on C003 so a future shared grounding owner or a C006 post-drift re-check is a planned lift, not a rediscovery. The full seam decision and its alternatives are in ADR015.

## Affects

- **Wiki Curation Architecture** — the Requirement-Component Map gains O009-R001 → C003, O009-R002 → C003, O009-R003 → C003; the C003 component summary and the Ingest/Query operation notes mention the grounding check and signal; ADR015 and this CR are added to See Also.
- **C003 - Integration Authoring** — intro claims O009-R001/R002/R003; data model, interfaces, behavior, edge cases, relationships, and success criteria gain the grounding capability, the distillation review, and the grounding-signal verb effect; `## Notes` records the delegation seam; See Also gains ADR015 and this CR.
- **C004 - OKF Conformance** — its owned-conventions set gains the per-claim grounding-signal marker (shape only); See Also gains ADR015 and this CR.
- **O009-R001 / O009-R002 / O009-R003** — now traced to their satisfying component in both directions.
- **PDR001 - Claim fidelity as a first-class outcome** — its "maps to no component" consequence is discharged by this trace and annotated accordingly.
- **ADR015 - Claim grounding seam** — the decision this change record realizes.
