# CR008 - Survey/Lint aggregator trace: C011

Gave the **Survey** and **Lint** operations an owning component. Added **C011 - Curation Operations**, a thin read-only aggregator over the detect faces, owning the face→verb routing table, the subject rollup rule, and the conflict-pairing rule — the cross-face aggregation logic that was previously unwritten skill prose with no owner or test. Updated the engineering README so Survey and Lint name C011 as their owner, added C011 to the Components list and to the O004-R003 row of the Requirement-Component Map, and updated the seven detect faces to name C011 as the aggregator they feed. The component-vs-contract decision is recorded with alternatives in [ADR016](../engineering/drs/ADR016-survey-lint-aggregation-ownership.md).

## Change

Before: Survey and Lint were skill-level aggregations of the detect faces with **no owning component** (engineering README, Operations: "Survey and Lint have no owning component"). Their cross-face logic — rolling up a concept surfaced by several faces into one item, de-duplicating facts C008 already folded into retirement candidates, pairing the expand-vs-retire conflict between C009's `thin` and C008's retirement candidacy, and routing each finding to a remediation verb — lived nowhere: no component owned it, no test covered it, and a new detect face had no doc to wire into.

After:

- **C011 - Curation Operations** (new component) owns the Survey and Lint operations. It exposes `survey(scope?) → survey_item[]` (fanning out to C006, C007, C008, C009, C010) and `lint(scope?) → lint_item[]` (fanning out to C004, C005, and the read-only index/recency freshness signal), grouping findings by subject, applying the rollup and conflict-pairing rules, and attaching routing. It performs no detection (each face keeps its judgment), writes nothing, and holds no OKF structural literal (it reads only through the faces). Its one owned artifact is the face→verb routing table; its two owned rules are the subject rollup and the conflict-pairing.
- **Requirement-Component Map** gains C011 on the **O004-R003 (assisted upkeep)** row, reciprocally with the faces it aggregates — C011 is the *surfacing* half's single view.
- **Engineering README Operations** section: the Survey and Lint paragraphs now name C011 as their owner; the "Survey and Lint have no owning component" sentence is removed and replaced with C011's ownership and the Ingest-parallel framing. C011 is added to the Components list.
- **C004, C005, C006, C007, C008, C009, C010**: each face's description of the Survey/Lint aggregation as a skill-level operation is updated to name C011 as the aggregator it feeds. No face changes what it detects or what requirement it claims — only the owner of the aggregation above it is named.

## Rationale

Ingest is a pipeline whose cross-stage logic is a trivial linear hand-off, so each stage being a deep component is enough. Survey's cross-face logic is not trivial: facts surface twice (C008 folds C006 drift and C007 staleness into its candidates), guidance contradicts (C009 `thin` vs C008 retirement on one concept, which C009 explicitly refuses to resolve), and a new face's wire-in is uncaught. That behavior needs an owner and a test surface. Leaving it as skill prose — the composition-contract alternative — resolves nothing testably; folding it into the skill is the status quo the finding flags. A thin aggregator component gives the rollup and conflict-pairing rules a legible owner and a forcing function ("every face is represented" fails if a face is added without wiring in), consistent with how this architecture owns all other behavior. The full decision and its alternatives are in ADR016.

C011 is deliberately thin — fan-out plus three owned things (routing table, rollup rule, conflict-pairing rule) — and holds no detection, so each face's judgment stays local and independently testable while the composition gets a home.

## Affects

- **Wiki Curation Architecture** — the Operations section names C011 as Survey/Lint's owner (the "no owning component" statement removed); the Components list gains C011; the Requirement-Component Map's O004-R003 row gains C011; See Also gains ADR016 and this CR.
- **C011 - Curation Operations** — the new component this change record realizes; owns the Survey/Lint aggregation, the face→verb routing table, and the rollup and conflict-pairing rules; claims O004-R003.
- **C004 / C005 / C006 / C007 / C008 / C009 / C010** — each names C011 as the aggregator its detect face feeds (Survey for C006/C007/C008/C009/C010; Lint for C004/C005; C007 feeds both); See Also gains this CR. No detection or requirement claim changes.
- **O004-R003 - Assisted upkeep** — now also satisfied, on the surfacing half, by C011's aggregated Survey/Lint view; traced in both directions.
- **ADR016 - Survey/Lint aggregation ownership** — the decision this change record realizes.
