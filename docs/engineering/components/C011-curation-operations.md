# C011 - Curation Operations

Realizes the two standing-health operations — **Survey** and **Lint** — by aggregating the components' detect faces into one deduplicated, conflict-aware findings view and routing each finding to the remediation verb the steward directs. It performs no detection of its own and writes nothing to the corpus.

C011 is the **operations aggregator** the curation Agent Skill's Survey and Lint operations are realized by — the standing-capability counterpart to the C001 → C002 → C003 Ingest pipeline. Each detect face owns one kind of judgment over the corpus: source drift (C006 - Source Revalidation), staleness and recent change (C007 - Currency Tracking), retirement candidacy (C008 - Lifecycle & Retirement), coverage gaps (C009 - Coverage Review), and scope drift (C010 - Charter) for Survey; structural conformance (C004 - OKF Conformance), referential integrity (C005 - Index & Navigation), and index/recency freshness (C005, C007) for Lint. C011 owns the *cross-face* concerns none of those faces can own alone, because each is about the **relationship between two faces' outputs**: fanning out to the faces over a scope, rolling their findings up so a concept surfaced by several faces reads as one coherent item rather than N disconnected ones, pairing findings that give the steward contradictory guidance, and routing each finding to the write verb that would resolve it. It is read-only and recommend-and-confirm, exactly like the faces it aggregates (intent in, work out): every finding is a candidate the steward acts on through a C003 or C008 verb, never a write C011 performs. As the one view the steward reads before directing a verb, C011 carries the *surface* half of assisted upkeep (O004-R003) — reciprocally with the detect faces it aggregates and the C003/C008 verbs that carry out the confirmed action — so the steward audits and edits nothing by hand. This gives the previously unwritten aggregation logic a single owner, a test surface, and the one place a new detect face must wire into (see [ADR016](../drs/ADR016-survey-lint-aggregation-ownership.md)).

## Data model

C011 owns **no persisted state** and **no detection**. Its entire output is a transient view recomputed from the faces on each run, mirroring the detect faces themselves. It owns one piece of standing data — the **routing table** — and the two rolled-up finding shapes.

- **Face→verb routing table** (the one owned artifact, a static map, not corpus state). One row per detect-face finding kind, naming the remediation verb(s) a finding of that kind routes to. This is the single place the finding→verb mapping lives, so a new detect face is wired in by adding its row (and a co-presentation rule if it can overlap an existing face):
  - drift (C006) → `C003.revise` (re-ground the claim against its moved source)
  - staleness (C007 recency) → `C003.revise` (refresh) or `C008.deprecate` / `delete` (retire, if obsolete)
  - retirement candidate (C008) → `C008.delete` / `deprecate` / `move` (the steward's free choice)
  - scope drift (C010 reconcile) → `C008.deprecate` / `delete` / `move`, or `C010.revise_charter` (widen the scope to include it)
  - coverage gap (C009) → `C003.create` (absent area) or `C003.revise` (thin concept)
  - structural finding (C004 validate) → `C003.revise`, or a mechanical re-render of a generated artifact
  - referential / dangling link (C005) → `C003.revise` (author the target or correct the link); a persistent in-scope dangler is also a C009 gap signal, so it may surface as a coverage finding instead
  - index/recency freshness (C005, C007) → a mechanical re-render, resolved as the terminal effect of the next write verb ([ADR011](../drs/ADR011-concept-verb-surface.md))
- **Survey item** (transient, never written to the corpus). The rolled-up unit the steward acts on — one per concept, or per gap `area` for coverage findings, never one per face:
  - **`subject`** — the concept-id, or the coverage `area` for an `absent` gap that names no existing concept.
  - **`findings: [{ face, kind, rationale, source_signals? }]`** — every face finding that named this subject, each keeping its own rationale. `source_signals` carries a retirement candidate's constituent signal list (C008 already fuses drift, staleness, and supersession) so those facts are represented once, inside the candidate, and not re-emitted as standalone raw findings for the same concept (see Behavior).
  - **`suggested_verbs`** — the union of the routing-table verbs for the subject's findings; the routes the steward may direct, never actions C011 takes.
  - **`conflict?`** — set when two of the subject's findings route to mutually exclusive actions (most sharply *deepen* vs *retire*); see Behavior.
- **Lint item** (transient, never written to the corpus). One per concept or path carrying at least one lint finding: `{ subject, findings: [{ face, kind ∈ { structural, referential, freshness }, severity?, rationale }], suggested_resolution }`. `severity` carries C004's `error` / `advisory` classification through unchanged.

## Interfaces

C011 exposes an in-process face. It reads the corpus only *through the detect faces* (which read through C004); it holds no direct path to corpus content and encodes no OKF structural literal. Nothing it exposes writes to the corpus.

- **`survey(scope?) → survey_item[]`** — the periodic standing-health sweep behind the product's **Survey** operation. Over `scope` (a directory; default the whole bundle) it calls `C006.revalidate_all`, `C007.recency` / `history_all`, `C008.retirement_candidates`, `C009.coverage_review`, and `C010.reconcile`, groups their findings by subject, applies the rollup and conflict-pairing rules, attaches the routing suggestions, and returns the items. Read-only with respect to the corpus. Degrades with its faces: a face that returns empty (e.g. `reconcile` / `coverage_review` with no charter) simply contributes nothing; C011 never errors on an empty face.
- **`lint(scope?) → lint_item[]`** — the standing-conformance sweep behind the product's **Lint** operation. Over `scope` it calls `C004.validate`, `C005.dangling_links`, and the read-only index/recency freshness signals C005 and C007 expose, groups by subject, and returns the items with their resolutions. Read-only — a Lint pass triggers no write, so it mutates nothing even when it reports staleness (it reports the freshness signal; it never calls a mutation to fix it).

## Behavior

- **Aggregate and route; never detect, never write.** Every judgment C011 returns was made by a face; every write a finding implies is a verb the steward directs. C011 adds only the cross-face view: fan-out, rollup, conflict-pairing, and routing. This keeps each face's judgment local (the one-owner discipline) while giving the composition a home.
- **Roll up by subject; state each fact once; union the routes.** A concept named by more than one face is one Survey item, not one per face. A fact that C008's `retirement_candidates` already folded in as a signal — the drift it read from C006, the staleness it read from C007 — is presented inside that candidate and *not* re-emitted as a separate raw C006/C007 item for the same concept, so the steward sees the drifted-and-a-retirement-candidate concept once, not three times. Raw C006 drift and C007 staleness surface as their own items only for concepts C008 did **not** roll into a candidate (e.g. a freshly drifted source on an otherwise-healthy concept, whose remedy is a `revise`, not a retirement). The item's `suggested_verbs` is the union of every constituent finding's routes, so no remediation path is dropped by the rollup.
- **Pair contradictory guidance; suppress nothing.** When one concept carries findings whose remediations are mutually exclusive — the sharp case is C009 marking it `thin` (route: `revise` to *deepen*) while C008 lists it as a retirement candidate (route: `delete` / `deprecate` to *retire*) — C011 sets `conflict` and presents both in the one item, so the steward reconciles *expand vs retire* explicitly instead of meeting "deepen this" and "retire this" as two unrelated items and never seeing they name the same concept. C011 suppresses neither side, matching C009's own posture; it is the place the two faces meet.
- **Survey is semantic and over-time; Lint is mechanical and right-now.** The two operations stay distinct in kind and fan out to different face sets: Survey aggregates the substance-and-currency faces (C006, C007, C008, C009, C010), whose findings need the steward's judgment; Lint aggregates the form-and-consistency faces (C004, C005, and the freshness signal), whose findings are mostly objective. Lint findings rarely name the same fact, so its rollup is light — but it groups by subject on the same rule, so a concept with both a structural advisory and a dangling link reads as one item.
- **Degrade with the faces, never below them.** C011 makes no independent claim; its output is exactly what its faces report, grouped. With no charter, the `reconcile` and open `coverage_review` signals are empty and Survey narrows to drift, staleness, retirement candidacy, and dangling-link gaps — the same clean degradation the faces define, surfaced through one view.

## Edge cases

- **A concept that is both `thin` (C009) and a retirement candidate (C008)**: one Survey item with `conflict` set, carrying both findings and both routes (`revise`-to-deepen and `delete` / `deprecate`) — the expand-vs-retire decision the steward reconciles. Neither face suppresses the other, and neither does C011.
- **A concept that is drifted (C006) and also a retirement candidate whose signals include that drift (C008)**: one item; the drift is shown once, as the candidate's `drift` signal, not also as a standalone C006 finding. `suggested_verbs` still offers both `revise` (re-ground) and the retirement verbs, so re-grounding remains available if the steward would rather keep the concept.
- **A concept drifted (C006) but not a retirement candidate**: surfaces as its own Survey item routing to `revise` — drift matters on its own, independent of retirement.
- **One missing target referenced from many concepts**: C009 already collapses it to one `absent` coverage finding; C011 surfaces that one finding as one item, keyed by `area`, not one per referrer.
- **A dangling link that C005 reports and C009 also surfaces as a gap**: presented as the C009 coverage item (the interpreted judgment), not duplicated as a raw C005 referential item — C009 is the consumer that judged the dangler, so its finding is the one C011 carries.
- **No charter declared**: `reconcile` and the open agent proposals within `coverage_review` return empty; Survey omits scope-drift items and returns the scope-independent findings only, without error.
- **A face raises an error or is unavailable**: C011 surfaces the faces that did return and reports the failed face as a degraded-coverage note rather than dropping it silently — a Survey that silently skipped a face would read as "nothing to do" when the face was simply not run.
- **`scope` narrower than the whole bundle**: each face is called with the same `scope`; items are limited to subjects within it. A cross-scope dangling link is reported per the face's own scoping.

## Relationships

- **C006 / C007 / C008 / C009 / C010**: the Survey detect faces C011 reads and aggregates — `revalidate_all`, `recency` / `history_all`, `retirement_candidates`, `coverage_review`, `reconcile`. C011 calls each read-only and depends on none of them calling it; the faces are unaware they are aggregated (no reverse dependency, no shared state). C008's `retirement_candidates` is itself a within-face aggregation (drift + staleness + supersession); C011's rollup composes *across* faces, on top of that, and de-duplicates the facts C008 already folded in.
- **C004 / C005**: the Lint detect faces — `C004.validate` (structural conformance) and `C005.dangling_links` (referential integrity), plus the read-only index/recency freshness signals C005 and C007 expose. C011 reads their findings; it reaches corpus content only through them, so it holds no OKF structural literal (the one-owner invariant, ADR001, stays clean with C011 present).
- **C003 - Integration Authoring** and **C008 - Lifecycle & Retirement**: the remediation verbs C011's routing table points a finding at — `create` / `revise` (C003) and `deprecate` / `delete` / `move` (C008). C011 names the route; it never calls the verb. The steward reads the Survey/Lint item and directs the verb (recommend-and-confirm), so no runtime call runs from C011 into a write path.
- **C001 - Ingestion Queue / C002 - Triage**: no relationship — those compose the Ingest pipeline (build and integrate new material), a different operation from the standing-health Survey and Lint sweeps C011 owns. C011 aggregates detection over the *existing* corpus; Ingest admits *incoming* material.
- **Boundary**: C011 owns cross-face aggregation for the Survey and Lint operations — fan-out, subject rollup, conflict-pairing, and the finding→verb routing table — and nothing else. It performs no detection (each face owns its judgment), authors or removes nothing (C003, C008), defines no format (C004), and owns neither Ingest (the C001 → C002 → C003 pipeline) nor Query (direct file read) — neither of which is a cross-face aggregation.

## Success criteria

- **Every face is represented**: for each detect face, a finding it reports over a scope appears in the corresponding `survey`/`lint` item for that scope — verifiable per face by seeding a condition it detects (a drifted source, a stale concept, a retirement candidate, a coverage gap, an out-of-scope concept; a structural error, a dangling link) and confirming it surfaces. This is the forcing-function test: a new detect face is not wired in until it has such a check, and an omitted wire-in fails it.
- **One item per subject**: a concept surfaced by more than one face appears exactly once in `survey` output, carrying every face's finding — verifiable by making one concept simultaneously drifted, stale, and out of scope and confirming a single item with three findings, not three items.
- **No duplicated fact**: a drift or staleness fact that C008 folded into a retirement candidate is not also emitted as a standalone raw finding for the same concept — verifiable by making a concept a retirement candidate on a drift signal and confirming the drift appears only within the candidate's signal list.
- **Conflict surfaced**: a concept that is both `thin` (C009) and a retirement candidate (C008) produces one item with `conflict` set and both routes present — verifiable by constructing that pair and confirming a single conflicted item.
- **Read-only**: `survey` and `lint` leave the corpus byte-for-byte unchanged on every call, and a Lint pass triggers no mutation even when it reports index/recency staleness — verifiable by diffing the corpus before and after.
- **Routes match the table**: each finding's `suggested_verbs` is exactly the routing-table entry for its kind — verifiable against the table.
- **No persisted state**: C011 stores no findings or dismissal artifact; a re-run with no corpus change reproduces the view from the faces alone.
- **One-owner invariant preserved**: C011 contains no OKF structural literal — ADR001's grep-able check stays clean with C011 present.

## Notes

- Survey and Lint were previously described as skill-level aggregations with **no owning component** (engineering README, Operations). Giving that aggregation an owner — rather than leaving the rollup, conflict-pairing, and routing rules as unwritten skill prose that no test covers and no doc forces a new face to wire into — is the decision recorded, with its alternatives (a composition-contract section, or folding the logic into the skill), in [ADR016](../drs/ADR016-survey-lint-aggregation-ownership.md). The name was chosen over the alignment review's "Curation Console" because C011 has no interactive surface — it is a read-only aggregation the Agent Skill calls, not a console.
- C011 depends only on a **read-only** freshness signal from C005/C007 for Lint; it never triggers `reindex` (a mutation) to check freshness, so a Lint pass mutates nothing. The exact read-only freshness interface is C005's and C007's to provide; C011 pins no mutation.
- The rollup and conflict-pairing rules encode editorial judgment about *presentation*, not detection: C011 changes what the steward sees as one item, never what a face judged. Each face's judgment stays local and testable in isolation; C011's tests cover only the composition.

## See Also

### Architectural Decision Records

- [ADR016 - Survey/Lint aggregation ownership](../drs/ADR016-survey-lint-aggregation-ownership.md)

### Change Records

- [CR008 - Survey/Lint aggregator trace: C011](../../crs/CR008-survey-lint-aggregator-trace.md)
