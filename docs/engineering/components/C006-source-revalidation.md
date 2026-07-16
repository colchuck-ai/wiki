# C006 - Source Revalidation

Detects when a cited source has changed since a claim was drawn from it — comparing the live source against the durable reference or captured snapshot — and reports drift for the steward rather than repairing it.

C006 is a **read-and-compare layer** over concepts C003 has already authored: it never authors, edits, or repairs a concept or its citations. It fulfills drift detection (O001-R003) outright and, as one of the corpus's detect faces, shares assisted upkeep (O004-R003) with the corpus's other detect faces and C003's remediation verbs — surfacing drift so the steward audits no source by hand. Through that same signal it feeds the undetected-staleness risk C001-RSK002 and O002-RSK001 name — its output is one of the measurable retirement signals C008 - Lifecycle & Retirement will later weigh (O003-R002). C006 is the only component that reaches outside the repository: revalidating a captured-snapshot citation means fetching its live `origin` URL, a mechanical byte comparison rather than the kind of judgment call C002's overlap scan or C010's reconciliation delegate to an agent pass. Like C005 and C010, C006 detects and reports; it repairs nothing itself.

C006 reads a citation's live locator and its drift baseline entirely off the authored concept and the repository — it has no runtime dependency on C001 - Ingestion Queue's task record after a concept is authored ([ADR007](../drs/ADR007-citation-link-form-for-drift-revalidation.md)).

## Data model

- **Drift finding** (transient, not written to the corpus). One per citation C006 checks: `{ concept-id, citation-number, source-kind ∈ { in-place, captured-snapshot }, verdict, rationale }`, where `verdict ∈ { unchanged, drifted, unreachable, not-revalidatable }`. Mirrors C005's inbound/dangling-link results and C010's drift finding: an answer to a query, never persisted.
- **Baseline**, computed on demand, never stored:
  - **Captured-snapshot citation**: the current bytes of the archived `/references/` asset the citation names as its durable fallback copy. Stable across calls because C001 guarantees the archived asset is immutable once captured.
  - **In-place citation**: the cited path's content as of the earliest commit that introduced the concept file, read via git history (`git log` / `git show` on the concept and the cited path) rather than any hash stored in the corpus. A citation edited without a corresponding content change to the concept still anchors to the concept's earliest commit — a known imprecision noted under Edge cases, not a correctness requirement C006 makes stronger claims about.
- C006 owns neither of these as a persisted artifact — both are recomputed fresh on every `revalidate` call, consistent with the on-demand-computation precedent [ADR006](../drs/ADR006-index-materialization-and-graph-computation.md) set for C005.

## Interfaces

C006 exposes an in-process face plus one live network read. Every corpus read goes through C004; nothing C006 exposes writes to the corpus.

- **`revalidate(concept_id) → drift_finding[]`** — checks every citation on one concept (one entry for a single-source concept; one per numbered citation for a multi-source merge) and returns a finding for each. For a captured-snapshot citation, fetches the live `origin` URL itself and hashes the response; for an in-place citation, reads the cited path's content at the concept's earliest commit via git and diffs against the path's current content. Read-only — no corpus write, ever.
- **`revalidate_all(scope?) → drift_finding[]`** — the whole-bundle (or `scope`-limited) sweep behind the product's **Survey** operation: runs `revalidate` over every concept with at least one citation.
- **`is_revalidatable(concept_id) → bool`** — true when the concept's citation(s) resolve to a locator C006 can actually check (a fetchable `origin`, or a resolvable in-place path with git history); false for a best-effort or descriptive-only citation (see Edge cases). Exposed so callers — including C008's `retirement_candidates` sweep — can distinguish "checked, unchanged" from "not checkable" without misreading the latter as a clean bill of health.

## Behavior

- **Mechanical comparison, not judgment.** Drift detection is a byte-level comparison (hash match / mismatch), not a semantic read of whether the change matters — that judgment is the steward's, working from the finding's rationale. This is why C006, unlike C002's overlap scan or C010's reconciliation, owns its live fetch directly rather than delegating to an agent pass: there is no interpretation step the fetch itself needs.
- **Baseline is always computed, never stored.** Re-running `revalidate` twice with no intervening change to either side yields the same `unchanged` verdict both times; nothing about the second run is faster or different because of the first — there is no cache to warm or invalidate.
- **Kind-specific comparison.** A captured-snapshot citation's live side is a fresh fetch of `origin`; a mismatch against the archived asset's current bytes is `drifted`. An in-place citation's live side is the cited path's current content in the same repository; a mismatch against that path's content as of the concept's earliest commit is `drifted`. C006 reads the citation through C004's parse/citation helper to tell which kind it is ([ADR007](../drs/ADR007-citation-link-form-for-drift-revalidation.md)) — it does not guess from the URL shape.
- **Report, never repair.** `revalidate` and `revalidate_all` only return findings. Correcting a drifted claim, re-citing a moved source, or removing a concept on persistent drift is the steward's judgment, executed through C003's `revise` or C008's `delete`/`deprecate` — not C006. This mirrors C004's, C005's, and C010's detect-don't-repair posture.
- **Unreachable is not drifted.** A live fetch that fails (network error, 404, timeout) yields `unreachable`, not `drifted` — C006 makes no claim about content it could not retrieve. Distinguishing "confirmed changed" from "could not confirm either way" is the point of separating these verdicts.

## Edge cases

- **Origin unreachable at revalidation time**: verdict `unreachable` with the fetch error as rationale — never reported as `drifted`, never silently dropped from the result set.
- **Non-text origin** (e.g. a PDF or image behind the citation): compared byte-for-byte against the archived binary asset; no text diff attempted.
- **In-place cited path deleted or moved since citation**: verdict `drifted` with a rationale distinguishing "target missing" from "content changed" — a citation-level concern distinct from C005's `dangling_links`, which tracks knowledge cross-links between concepts, not citations to material outside the concept graph.
- **Concept with a capture-failed or descriptive-only citation** (C001's origin-unreachable-at-intake case, or C003's best-effort citation for a capture-failed admitted task): `is_revalidatable` returns false and `revalidate` reports `not-revalidatable` rather than fabricating a comparison against nothing.
- **Multi-source (merged) concept**: each numbered inline citation is revalidated independently; one drifted source does not block a finding on the others.
- **Concept authored with no `# Citations` block**: `revalidate` returns an empty finding list — nothing to check, not an error.
- **Citation edited without the concept's content otherwise changing** (in-place case): the git-history baseline still anchors to the concept file's earliest commit, so a citation swapped in later is checked against a baseline older than the claim it actually backs — a known imprecision; see Notes.

## Relationships

- **C001 - Ingestion Queue**: none at runtime. C001's own document anticipated C006 reading `origin` and `content-hash` from its task record; that dependency is resolved without one — C006 reads everything it needs from the authored concept instead ([ADR007](../drs/ADR007-citation-link-form-for-drift-revalidation.md)), matching the self-contained precedent C002 and C003 set with C005 (ADR004, ADR005).
- **C003 - Integration Authoring**: the writer whose citation contract C006 depends on — a captured-snapshot citation's live `origin` link and archived-asset fallback, an in-place citation's unchanged repo path. No runtime call between them; C006 only reads what C003 already wrote.
- **C004 - OKF Conformance**: C006's sole path to corpus content — the parse helper to read a concept's `# Citations` block and tell a captured-snapshot citation from an in-place one. C006 encodes no OKF structural literal itself.
- **C005 - Index & Navigation**: no relationship. C005's `dangling_links` covers knowledge cross-links between concepts; C006's drift covers citations to material a claim was drawn from, in or outside the corpus. The two never check the same link.
- **C008 - Lifecycle & Retirement**: consumes C006's drift findings as one retirement-candidate signal (O003-R002, "source drift") alongside staleness (C007) and supersession, in `retirement_candidates`, as [ADR006](../drs/ADR006-index-materialization-and-graph-computation.md) already anticipated for C005's `dangling_links`.
- **C010 - Charter**: none. The charter concept carries no external citation, so it is never a candidate for revalidation — a natural consequence of C006's contract, not an exclusion rule C006 has to encode.
- **Boundary**: C006 owns citation drift detection only — comparing a citation's live locator against its baseline and reporting the result. It does not repair a citation, re-author a concept (C003), decide what to delete or deprecate (C008), maintain the cross-link graph (C005), or judge scope (C002, C010).

## Success criteria

- **Drift detected**: a live source whose content no longer matches the baseline yields a `drifted` finding — verifiable by revalidating, then changing the live-equivalent fixture, then revalidating again and observing the verdict flip.
- **No false positives**: an unchanged live source and an unchanged in-place path yield `unchanged` on every call.
- **Failure is distinct from drift**: an unreachable origin never produces a `drifted` verdict; it produces `unreachable` with the fetch error retained as rationale.
- **Non-revalidatable is distinct from clean**: a capture-failed or descriptive-only citation reports `not-revalidatable`, never `unchanged`.
- **No corpus mutation**: `revalidate`, `revalidate_all`, and `is_revalidatable` leave the corpus byte-for-byte unchanged on every call — verifiable by diffing the corpus before and after.
- **Stateless recomputation**: running `revalidate` twice with no intervening change on either side produces identical findings both times, with nothing persisted between calls.

## Notes

- The git-history baseline for in-place citations (reading the cited path's content as of the concept's earliest commit, rather than a stored hash) is a C006-internal implementation choice — it needs no format change to C003 or C004, so it is recorded here rather than in a separate ADR. Its known imprecision (a citation swapped in after the concept's first commit anchors to an older baseline than the claim it backs) is acceptable in the low-volume, single-steward envelope; tightening it to per-citation-edit granularity is a future optimization, not a requirement this component makes today.
- The captured-snapshot citation link form C006 depends on — the live `origin` URL as the primary citation link, with the archived asset as fallback — was settled specifically to give C006 a self-contained contract; see [ADR007](../drs/ADR007-citation-link-form-for-drift-revalidation.md) and [CR001](../../crs/CR001-external-citations-cite-live-origin.md).
- Revalidation is surfaced through the product's **Survey** operation (engineering README, Technology Choices): the tool runs `revalidate_all` and presents drift findings for the steward's judgment.

## See Also

### Architectural Decision Records

- [ADR007 - Citation link form for drift revalidation](../drs/ADR007-citation-link-form-for-drift-revalidation.md)

### Change Records

- [CR001 - External citations cite the live origin](../../crs/CR001-external-citations-cite-live-origin.md)
- [CR006 - Assisted-upkeep traceability: C003 and reciprocal claims](../../crs/CR006-assisted-upkeep-traceability.md)
