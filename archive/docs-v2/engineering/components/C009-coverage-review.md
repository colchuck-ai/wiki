# C009 - Coverage Review

Surfaces areas within the intended scope that are thin or absent — from dangling links, the charter's declared scope, and an agent scan of the corpus — as a sourcing agenda for the steward, so gaps do not persist unnoticed.

C009 is a **detect-don't-repair standing capability**, the completeness counterpart to C010's on-scope reconciliation: where C010 - Charter finds content that has drifted *outside* the declared scope, C009 finds scope territory the corpus *has not yet reached*. It reads three gap signals — dangling links (C005 - Index & Navigation), the declared scope (C010 - Charter), and its own agent scan of the corpus through C004 - OKF Conformance — clusters them into one gap agenda, and routes each confirmed gap to a write verb the steward directs (`create` for an absent area, `revise` to deepen a thin one, both on C003 - Integration Authoring). It writes nothing to the corpus itself. It fulfills coverage review (O007-R002) outright and shares assisted upkeep (O004-R003) with the corpus's other detect faces and C003's remediation verbs; it is the guard for silent gaps (O007-RSK002). Its agent scan is self-contained, following the precedent [ADR004](../drs/ADR004-triage-disposition-model.md) and [ADR005](../drs/ADR005-integration-authoring-provenance-and-merge.md) set — no dependency on C002's overlap machinery. The signal model, no-charter degradation, and scope-anchored suppression are captured in [ADR013](../drs/ADR013-coverage-review-signal-model.md).

## Data model

C009 owns **no persisted state** — no coverage store, no dismissal list. Its entire output is a transient agenda recomputed from its inputs on each run, mirroring the detect faces of C006 and C010. A deliberate exclusion is not C009 state; it lives in the charter's `# Out of scope` (C010), which C009 reads and respects (see [ADR013](../drs/ADR013-coverage-review-signal-model.md)).

- **Coverage finding** (transient, never written to the corpus). One per gap area the sweep surfaces:
  - **`area`** — a short description of the missing or under-covered territory, concrete enough for the steward to direct a `create` or `revise`.
  - **`kind ∈ { absent, thin }`** — `absent` when no concept covers the area; `thin` when a concept exists but under-covers it.
  - **`existing_concept_id?`** — set only for `thin`: the under-covering concept the steward would `revise` to deepen. Absent for `absent` gaps.
  - **`referenced_paths?`** — for an `absent` gap derived from dangling links, the not-yet-resolving target path(s) that referring concepts already point at.
  - **`signals: [{ source ∈ { dangling-link, charter-scope, agent-scan }, rationale }]`** — one entry per signal that surfaced the area. An area can carry more than one source (e.g. a dangling link *and* an agent proposal); the signals are not collapsed into a single verdict, the same multi-signal shape as C008's retirement candidate.

## Interfaces

C009 exposes an in-process face. Every corpus read goes through C004; the corpus is otherwise untouched (the sweep is read-only).

- **`coverage_review(scope?) → coverage_finding[]`** — the periodic whole-corpus sweep behind the product's **Survey** operation. Over `scope` (a directory; default the whole bundle) it gathers the three signals, clusters them by area, filters out areas the charter places out of scope and the charter concept itself, and returns the remaining gaps with their rationale. Read-only with respect to the corpus. Degrades cleanly when no charter is declared: the `charter-scope` signal is empty and open agent proposals are suppressed, but `dangling-link` gaps still surface (see Behavior). Never `create`s or `revise`s anything — every finding is a candidate for the steward.

## Behavior

- **Three signals, one agenda.** `coverage_review` fuses three heterogeneous signals into one clustered list:
  - **Dangling links** (from `C005.dangling_links`) — a link whose target does not resolve is authored intent to a concept that does not exist. C005 reports these but explicitly defers the judgment of *why* a link dangles to its consumer; C009 is that consumer. A surviving dangling link is, in steady state, a deliberate forward reference (orphan-from-removal links do not persist — C008 repairs them), so a persistent one that points into scope is a real `absent` gap.
  - **Charter scope** (from `C010.get_charter`) — the declared `# Purpose` and `# In scope` boundaries name territory the corpus is meant to cover; the agent compares that declaration against what the corpus actually holds and proposes the unreached parts.
  - **Agent scan** — C009's own open pass over the corpus through C004, proposing thin or absent areas the team likely relies on that neither of the other two signals named.
- **Detect, don't repair; recommend, never author.** `coverage_review` only surfaces candidates. Filling an `absent` gap is `C003.create`; deepening a `thin` one is `C003.revise` — both the steward's directed action. C009 never writes a concept, exactly the posture of C006 and C010.
- **Interpret dangling links; C005 does not.** C005 reports every unresolved link without judging it (a forward reference and an orphan look identical to C005). C009 applies the judgment: a dangling target that is in scope and substantial becomes an `absent` gap; one that is out of scope or trivial is filtered out and never surfaced.
- **Cluster and de-duplicate by area.** Many dangling links to the same missing target collapse to one finding; an agent proposal that overlaps a dangling-link gap becomes one finding carrying both a `dangling-link` and an `agent-scan` signal. The steward sees one item per area, not one per signal.
- **Thin versus absent.** `absent` means no concept covers the area at all; `thin` means a concept exists but under-covers its area, and the finding names it via `existing_concept_id`. Thinness is the agent scan's content judgment about *coverage depth of the standing corpus* — deliberately distinct from C002's significance bar (which judges *incoming* material at intake) and from C008's retirement signals (which judge whether to *remove* a concept). C009 only ever judges that an area needs more, never that a concept should go.
- **Scope-anchored; degrades cleanly without a charter.** With no charter declared, the `charter-scope` signal is empty and open agent proposals are suppressed — there is no declared scope to anchor "intended" against, so speculating about what is missing would be baseless. Dangling-link gaps still surface, because authored intent to a concept is meaningful regardless of whether scope was declared. C009 therefore never errors on a missing charter; it simply narrows to the one scope-independent signal (see [ADR013](../drs/ADR013-coverage-review-signal-model.md)).
- **Deliberate exclusions run through the charter.** When the steward decides an area is intentionally *not* covered, they add it to the charter's `# Out of scope` (`C010.revise_charter`); C009 reads that boundary and never surfaces a gap within it again — even one backed by dangling links. C009 keeps no dismissal store of its own, so scope stays a single authority (ADR003) and no C009 state can drift from the charter ([ADR013](../drs/ADR013-coverage-review-signal-model.md)).
- **Charter is meta.** The charter concept describes the corpus; it is not corpus content to be covered, so `coverage_review` never proposes it as a gap — the exclusion C010 relies on C009 to apply.

## Edge cases

- **No charter declared**: `coverage_review` returns only `dangling-link` gaps; the `charter-scope` signal is empty and open agent proposals are suppressed. Unlike `C010.reconcile` (which returns empty with no charter), C009 still produces the scope-independent signal.
- **Dangling link to an out-of-scope or trivial target**: not a gap — filtered by the charter's scope and the significance judgment, and never surfaced.
- **Dangling link that is a genuine forward reference to a wanted concept**: surfaced as an `absent` finding carrying the referring concept(s) in `referenced_paths` — the intended case for the dangling-link signal.
- **Area listed in the charter's `# Out of scope`**: never surfaced, even when dangling links point into it — the steward's deliberate exclusion is honored.
- **One missing target referenced from many concepts**: one `absent` finding, its `dangling-link` signal naming the referrers; not one finding per referrer.
- **An agent proposal that overlaps a dangling-link gap**: merged into a single finding with both a `dangling-link` and an `agent-scan` signal, not two findings.
- **Charter with an empty or placeholder `# Purpose`**: cannot anchor a scope judgment; C009 falls back to dangling-link gaps only and surfaces the weak-charter condition (consistent with C010's handling) rather than emitting baseless `charter-scope` gaps.
- **A concept that is both thin (C009) and a retirement candidate (C008)**: C009 assesses area coverage independent of retirement candidacy, so it may surface the concept as `thin` while C008 surfaces it as stale or drifted; expanding it versus retiring it is opposite guidance the steward reconciles, and C009 suppresses neither.

## Relationships

- **C005 - Index & Navigation**: reads `dangling_links` as the `dangling-link` signal and supplies the interpretation C005 defers — deciding which unresolved links represent real in-scope gaps. No reverse dependency: C005 reports links without knowing C009 consumes them.
- **C010 - Charter**: reads `get_charter` for the `charter-scope` signal, respects its `# Out of scope` as the sole gap-suppression mechanism, and excludes the charter concept from gaps. C009 is the completeness counterpart to `C010.reconcile` — reaching-into-scope versus drifted-out-of-scope — both adjudicating against the one declared charter, so scope stays single-authority (ADR003, [ADR013](../drs/ADR013-coverage-review-signal-model.md)).
- **C003 - Integration Authoring**: the write verbs every confirmed gap routes to — `create` for an `absent` area, `revise` to deepen a `thin` one. C009 recommends; C003 authors. The routing is the steward's directed action through Survey, not a runtime call from C009 into C003 (recommend-and-confirm; intent in, work out).
- **C004 - OKF Conformance**: sole path to corpus content for the agent scan — parsing concepts and frontmatter through C004's helpers. C009 encodes no OKF structural literal.
- **C002 - Triage**: no relationship — C002 judges the significance and scope of *incoming* material at intake; C009 judges the *completeness* of the standing corpus. Distinct passes over distinct material, the same boundary C008 draws with C002.
- **C006 / C007 / C008**: co-detect faces C011 - Curation Operations aggregates alongside C009 behind the **Survey** operation; C009 calls none of them and shares no state. C009's `thin` judgment is independent of C008's retirement candidacy (see Edge cases); when both name one concept, C011 co-presents them as the expand-vs-retire choice C009 leaves to the steward, suppressing neither ([ADR016](../drs/ADR016-survey-lint-aggregation-ownership.md)).
- **C011 - Curation Operations**: reads `coverage_review` as the coverage-gap face of its Survey aggregation and routes each finding to `create` (absent) or `revise` (thin); C011 aggregates and routes, C009 judges. No reverse dependency — C009 is unaware it is aggregated.
- **Boundary**: C009 owns aggregating gap signals into a sourcing agenda and judging which represent real in-scope gaps. It does not author or deepen concepts (C003), declare or reconcile scope (C010), define the OKF format (C004), or judge material at intake or removal (C002, C008). It writes nothing to the corpus and keeps no persisted state.

## Success criteria

- **Silent-gap guard**: a concept referenced by a surviving dangling link but absent from the corpus appears as an `absent` coverage finding — verifiable by adding a forward-reference link to a non-existent, in-scope concept and confirming it surfaces.
- **Detect, don't repair**: `coverage_review` never writes to the corpus; its entire output is candidates — verifiable by running it and confirming the corpus is byte-for-byte unchanged.
- **Scope-respecting**: an area named in the charter's `# Out of scope` never surfaces as a gap, even when dangling links point into it.
- **Clean degradation**: with no charter, `coverage_review` still returns `dangling-link` gaps and never errors; the `charter-scope` and open `agent-scan` proposals are simply empty.
- **One finding per area**: multiple signals for one area produce a single finding carrying all of them, never duplicate findings — verifiable by pointing a dangling link and an agent proposal at the same area and confirming one finding with two signals.
- **Charter excluded**: the charter concept never appears as a coverage gap.
- **No persisted state**: C009 stores no coverage or dismissal artifact — verifiable in that a re-run with no corpus or charter change reproduces the agenda from the inputs alone.
- **One-owner invariant preserved**: C009 contains no OKF structural literal — ADR001's grep-able check stays clean with C009 present.

## Notes

- Coverage review is surfaced through the product's **Survey** operation, which C011 - Curation Operations owns — C011 aggregates `coverage_review` alongside `C006.revalidate_all`, `C007.recency` / `history_all`, `C008.retirement_candidates`, and `C010.reconcile` (see [ADR016](../drs/ADR016-survey-lint-aggregation-ownership.md)).
- The three-signal aggregation model, the no-charter degradation, and the scope-anchored suppression (no C009-owned dismissal store) are captured — with their rejected alternatives — in [ADR013](../drs/ADR013-coverage-review-signal-model.md). The `thin`-or-`absent` scope follows O007-R002's wording directly and needs no separate decision.
- The self-contained agent scan follows the same self-contained precedent [ADR004](../drs/ADR004-triage-disposition-model.md) and [ADR005](../drs/ADR005-integration-authoring-provenance-and-merge.md) set for C002 and C003.

## See Also

### Architectural Decision Records

- [ADR013 - Coverage review signal model and scope-anchored gap suppression](../drs/ADR013-coverage-review-signal-model.md)
- [ADR003 - Charter as an in-corpus concept and single scope authority](../drs/ADR003-charter-as-in-corpus-concept.md)

### Change Records

- [CR006 - Assisted-upkeep traceability: C003 and reciprocal claims](../../crs/CR006-assisted-upkeep-traceability.md)
- [CR008 - Survey/Lint aggregator trace: C011](../../crs/CR008-survey-lint-aggregator-trace.md)
