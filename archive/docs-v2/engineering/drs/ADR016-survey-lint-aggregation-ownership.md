# ADR016 - Survey/Lint aggregation ownership

Give the **Survey** and **Lint** operations an owning component — **C011 - Curation Operations**, a thin, read-only aggregator over the detect faces — rather than leaving their cross-face aggregation as skill-level prose. C011 owns the concerns no single detect face can: the face→verb routing table, the subject rollup rule (deduplicating facts C008 already folded into retirement candidates), and the conflict-pairing rule (co-presenting a concept that is both `thin` and a retirement candidate). Detection stays in the faces; C011 adds only the composition, a test surface for it, and the single point a new face wires into.

## Context

The engineering architecture describes four steward-facing operations. **Ingest** is realized by the C001 → C002 → C003 pipeline — each stage a deep component, the cross-stage logic a trivial linear hand-off. **Query** is direct file read. But **Survey** and **Lint** were described as *skill-level aggregations of the detect faces, with no owning component* (README Operations, prior wording: "Survey and Lint have no owning component"). Their cross-face logic is not a trivial hand-off:

- **Facts surface twice.** `C008.retirement_candidates` already fuses drift (from C006) and staleness (from C007) into its candidate signals. A naive Survey that concatenates `C006.revalidate_all` + `C007.recency` + `C008.retirement_candidates` shows a drifted, stale retirement candidate as three disconnected findings. Something must roll them up by concept and state each fact once — and neither C006, C007, nor C008 can own that rule, because each is blind to how its output composes with the others.
- **Guidance contradicts.** `C009.coverage_review` can mark a concept `thin` (expand it) while `C008.retirement_candidates` lists the same concept for retirement (remove it). C009's own spec says it "suppresses neither" and defers the expand-vs-retire reconciliation to the steward — but nothing pairs the two, so the steward can meet "deepen this" and "retire this" as unrelated items and never see they name one concept.
- **New faces rot silently.** Adding a detect face has no doc that must be edited and no test that fails if the wire-in is forgotten — the scatter that leaves aggregation logic drifting out of step with the faces it aggregates.

So the question is narrow: **where does the Survey/Lint cross-face aggregation live, and how much of it gets a named owner and a test surface?** The routing table, the rollup rule, and the conflict-pairing rule are real behavior; the only open decision is whether they are owned by a component, a documented contract, or nothing.

## Options

- **A thin aggregator component, C011 - Curation Operations (chosen).** A read-only component with `survey(scope?)` / `lint(scope?)`, owning the face→verb routing table, the subject rollup rule, and the conflict-pairing rule. Deep enough to justify a component *because* the rollup and conflict rules are genuine behavior no single face can own — each is about the relationship between two faces' outputs — and because a component gives that behavior a test surface (`survey`/`lint` output is assertable) and a forcing function (a new face is not wired in until it has a "this face is represented" check). Consistent with how this architecture owns every other unit of behavior: in a component, not in prose. Cost: one more component, and a risk it reads shallow if under-loaded — mitigated by loading it with the two named rules and nothing that belongs to a face.
- **A composition-contract section in the engineering README (no component).** Keep Survey/Lint as skill-level operations, but write the routing table, rollup rule, and conflict-pairing rule into a "Survey/Lint composition" README section with success criteria. Lighter, and keeps the clean "operations are skill-level, components are capabilities" split. Rejected: architecture prose has no test surface and no single owner, so the rules that a passing test would pin stay unenforced — the rot risk restated, not resolved. The forcing function degrades to "remember to update the prose," exactly what failed here.
- **Fold the aggregation into the curation Agent Skill with no named owner.** Simplest — leave it where it is, as skill glue. Rejected: this is the status quo the finding flags. The rollup, conflict, and routing rules stay unwritten and untested, a new face's wire-in is uncaught, and the expand-vs-retire conflict has no home — the same "no legible owner to test or share" failure ADR015 rejected for the grounding judgment.
- **Extend an existing detect face to own the aggregation.** Have one face (e.g. C008, which already fuses three signals) aggregate the others. Rejected: no face's boundary covers all of Survey (C008 owns retirement, not coverage or scope), and making one face depend on the rest inverts the self-contained-face discipline (ADR004/ADR005) and creates the cross-face coupling the architecture avoids. The aggregation is a distinct concern from any single face's judgment.

## Decision

Add **C011 - Curation Operations**, a thin read-only aggregator, as the owner of the Survey and Lint operations:

- `survey(scope?) → survey_item[]` fans out to C006/C007/C008/C009/C010, groups findings by subject, applies the rollup and conflict-pairing rules, and attaches routing. `lint(scope?) → lint_item[]` fans out to C004/C005 and the read-only freshness signal, groups by subject, and attaches resolutions. Both are read-only; a finding's remediation is always a C003/C008 verb the steward directs, never a call C011 makes.
- C011 owns exactly three things no face can: the **face→verb routing table** (the single place the finding→verb map lives, and the row a new face must add), the **rollup rule** (one item per subject; a fact C008 already folded into a retirement candidate is not re-emitted raw; routes are unioned), and the **conflict-pairing rule** (a `thin`-and-retirement-candidate concept is one item with both routes and a `conflict` flag).
- C011 performs **no detection** and holds **no OKF structural literal** — it reads the corpus only through the faces (which read through C004), so ADR001's one-owner invariant stays clean. Each face's judgment stays local and independently testable; C011's tests cover only the composition.

C011 maps to **O004-R003 (assisted upkeep)**, reciprocally with the faces it aggregates: it is the *surfacing* half of "surface … and carry out … no hand-auditing or hand-editing," realized as the one view the steward reads before directing a verb.

The name is **Curation Operations**, not the alignment review's "Curation Console": C011 has no interactive surface — it is a read-only aggregation the Agent Skill calls.

## Consequences

- Survey and Lint gain a single owner, a test surface, and a forcing function: the "every face is represented" success criterion fails if a new detect face is added without wiring it in, curing the class of rot the finding named, not just the instance.
- The rollup and conflict-pairing rules — previously unwritten skill prose — are now specified and testable in one place. The double-surfacing of drift/staleness and the unowned expand-vs-retire conflict both have a defined resolution.
- The component set grows by one. C011 is deliberately thin (fan-out + three rules); the risk it reads as a shallow pass-through is accepted and bounded by keeping all detection in the faces and loading C011 only with cross-face behavior.
- The architecture's "operations are skill-level" description is refined: Ingest remains a pipeline of components with trivial glue, while Survey and Lint are realized *by* an aggregator component (C011) — the standing-health counterpart to the Ingest pipeline. Query stays direct read (no aggregation, no component).
- A new detect face now has a defined wire-in cost: add its routing-table row, a co-presentation rule if it can overlap an existing face, and a representation success criterion. This is a small, explicit tax that replaces silent omission.
- C011 depends only on read-only freshness signals from C005/C007 for Lint and pins no mutation, so it composes cleanly with a future tightening of C005's freshness interface (the separately-tracked read-only-freshness concern) without a contract change here. That concern is now resolved: `C005.index_status` and `C007.recency_status` supply the read-only interface, and C011's Lint reads them in place of any mutation ([CR010](../../crs/CR010-read-only-freshness-interface.md)) — exactly the clean compose this consequence anticipated.

## Affected elements

- **C011 - Curation Operations** — the new component this decision creates; owns the Survey/Lint aggregation, the routing table, and the rollup and conflict-pairing rules. Backlinks here.
- **Wiki Curation Architecture** — the Operations section is rewritten so Survey and Lint name C011 as their owner (the "no owning component" statement is removed); the Components list gains C011; the Requirement-Component Map's O004-R003 row gains C011. Recorded in [CR008](../../crs/CR008-survey-lint-aggregator-trace.md).
- **C004, C005, C006, C007, C008, C009, C010** — each detect face's description of the Survey/Lint aggregation as skill-level is updated to name C011 as the aggregator it feeds. Recorded in [CR008](../../crs/CR008-survey-lint-aggregator-trace.md).
