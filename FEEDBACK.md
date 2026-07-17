# Intent Alignment Feedback — Tracking

Findings from an `/intent` alignment check of the docs tree, run through three
perspectives (user/steward, maintainer/engineering, seam/traceability) plus the
mechanical linter. This file tracks progress so work can resume across sessions.

**Status legend:** `[ ]` todo · `[~]` in progress · `[x]` done · `[-]` won't do (with reason)

**How to apply:** each item is a modification, so it runs through the `/intent`
draft → judge loop (`python3 scripts/lint_intent_tree.py docs/` first, then the
binary checklist). Map/requirement edits are **updates** → warrant a CR. Adding a
new element (e.g. O009) is a **create** → no CR. Decisions-with-alternatives →
consider a PDR/ADR.

**Baseline:** mechanical linter passes clean (0 errors / 0 warnings, 29 files).
All problems below are semantic. `⚑` = independently raised by two lenses (high confidence).

---

## Tier 1 — Highest leverage (fix first)

### [x] T1.1 — [Product · critical] Missing outcome: claim *fidelity/correctness*
- **Done (2026-07-16, uncommitted):** Added O009 - Faithful representation (2 risks, 3
  requirements, risk-requirement map) inline on `docs/product/README.md`; amended the spine
  sentence to include *faithful to what that source says*; authored `PDR001-claim-fidelity-outcome.md`
  (fidelity as its own outcome vs. folding into O001). Linter clean (30 files, 0/0); binary
  checklist passed. Follow-up noted in the PDR: O009-R001/R002/R003 map to no component yet —
  needs an engineering trace to assign owners.
- **Problem:** Nothing in O001–O008 measures whether an authored claim faithfully
  represents what its source says. Wiki is a distiller ("every claim *it writes*",
  O001-R001); distortion-in-distillation is its most likely failure. O001 only gives
  a re-verify link; O001-R003 only catches the *source* changing *later*. Without a
  fidelity outcome, consumers who can't trust the distillation route questions back
  to the steward — collapsing the core job promise.
- **Location:** gap across outcome set; spine sentence `docs/product/README.md:3`.
- **Fix:** Add outcome **O009 - Faithful representation** — "Maximize the proportion
  of claims that accurately represent what their cited source supports." Risks:
  distortion-in-distillation; indistinguishable grounding. Requirements: source-grounded
  authoring check (flag unsupported claims for steward); grounding signal surfaced to
  consumers; distillation review (claim shown alongside source span at authoring).
  Amend README:3 so "trustworthy" includes *faithful to its source*. Reuses O001-R002's
  durable snapshot.
- **Type:** create (no CR). Consider a PDR (this is a real product decision).

### [x] T1.2 — [Engineering · buildability] `merge` verb cannot be built as specified
- **Done (2026-07-16, uncommitted):** Authored `ADR014-merge-composition-integrity.md` (two decisions
  with alternatives: `content_preserved` flag on `delete` vs. `is_merge` flag vs. separate post-`delete`
  review query; one-commit-or-none atomicity vs. accept-window-and-Lint vs. `merge` as a fourth primitive)
  and `CR005-merge-composition-integrity.md`. **Problem A:** added `content_preserved=false` to
  `C008.delete`; review-candidate suppression now derives from that truthful flag (merge passes `true`,
  supersession leaves `false`), so there is no merge-only branch and C003's pure-composition criterion
  narrows to *same corpus bytes + same repaired links (+ same-flag review report)*. **Problem B:** `merge`
  now carries an all-or-nothing contract — its `revise`+`delete` steps run in one transaction (one git
  commit or none), so a `delete` failure after the fold can't leave a silent duplicate; ADR011's per-verb
  atomicity is refined so atomicity *composes*. Edited C003, C008, README principle; added forward
  "Extended by ADR014" banners to ADR011 and ADR012. Linter clean (32 files, 0/0); binary checklist passed.
- **Problem A:** C003 claims `merge` is a *pure composition* = `revise(into, from)` +
  `delete(from, replacement=into)`, "no merge-only code path," verifiable by comparing
  the two. But standalone `delete` surfaces `review_candidates` while the merge step must
  *suppress* them — and `delete(id, [replacement])` has no parameter to express the
  difference. The spec's own verification test would fail.
- **Problem B:** ADR011 guarantees every verb atomic "including effects," but `merge`
  spans C003 (`revise`) then C008 (`delete`) with no composition-atomicity contract. If
  `delete` fails after `revise` succeeds → content folded into `into` *plus* original
  `from` still present = silent duplicate (the exact O003-RSK003 the verb prevents),
  created by the tool.
- **Location:** `C003-integration-authoring.md:71,24`; `C008-lifecycle-retirement.md:23,45`;
  `drs/ADR011`, `drs/ADR012`.
- **Fix:** Narrow "pure composition" to *corpus bytes* ("merge yields same bytes as
  revise+delete; the review report differs"); give `delete` an explicit
  `content_preserved`/`review` flag that `merge` passes (update the signature); state
  merge = one git commit or none.
- **Type:** update → CR + likely an ADR amendment. Good `/codebase-design` candidate.

### [x] T1.3 — [Traceability · ⚑] O004-R003 map row broken; CR003 never propagated
- **Done (2026-07-16, uncommitted):** Chose *reciprocate*, not *narrow* — O004-R003 is a two-part
  requirement ("surface … *and* carry out … no hand-auditing *or* hand-editing"), so the true
  satisfying set spans both halves: the five detect/lifecycle components (surface) **plus C003**
  (carry-out). Added C003 to the map row (`docs/engineering/README.md`); made every mapped
  component claim O004-R003 in its own intro — C003 as the remediation half, C006/C007/C009/C010
  as detect faces, C008 on both sides. Replaced C009's fragile sibling enumeration ("with C006,
  C007, C008, and C010" — the exact list that had rotted) with one uniform role-based phrase
  ("the corpus's other detect faces and C003's remediation verbs"), curing the class of defect,
  not just the instance. Authored `CR006-assisted-upkeep-traceability.md`; added CR006 backlinks
  to all six components + README (created Change Records sections for C009/C010). Linter clean
  (33 files, 0/0); binary checklist passed. Tier 1 now complete.
- **Problem:** Requirement-Component Map maps O004-R003 to {C006,C007,C008,C009,C010},
  but four of those five never claim O004-R003, and **C003 is absent** — even though CR003
  made C003.`revise`-from-steward-direction *the* assisted-upkeep interface. A recorded
  change (CR003) never reached the traceability artifact (the map). Bidirectionality fails
  both ways.
- **Location:** `docs/engineering/README.md:115`; component docs; `docs/crs/CR003-concept-verb-surface.md`.
- **Fix:** Add C003 to the O004-R003 row (the `revise` remediation path). Then either have
  C006/C007/C008/C010 reciprocate the claim (as C009 already does) or narrow the row to the
  components that actually claim it.
- **Type:** update → CR.

---

## Tier 2 — Significant

### [x] T2.1 — [Engineering · ⚑] ADR006 & ADR008 silently superseded by ADR011 (no banner)
- **Done (2026-07-17, uncommitted):** Added the `> **Superseded in part by [ADR011]…**` banner to both
  ADR006 (caller-orchestrated `reindex` → verb-internal effect; index-storage decision itself retained) and
  ADR008 (terminal `record` → verb-internal effect; `history(concept_id)` per-directory → bundle-wide; the
  materialization decision itself retained). Annotated ADR008's stale third Consequence inline
  (`**— Reversed by [ADR011]:**`) — kept the still-true physical fact (entries stay behind in the old
  directory's `log.md`, never moved) and reversed only the now-false query claim ("will not surface
  pre-relocation entries"). Closed the reciprocal side: added ADR006 and ADR008 to ADR011's *Affected
  elements* (ADR011 already linked ADR008 at line 17 but never ADR006, so the forward banner would have
  broken bidirectionality). **Bonus:** a tree sweep for the same stale framing found a live contradiction at
  `C005-index-navigation.md:39` — an edge case still naming "a writer bypassed the [reindex] contract" and
  claiming "Lint/Survey passes that call `reindex`," which contradicted both ADR011's internal-effect model
  *and* C005's own line 51 ("Lint never calls `reindex`… mutates nothing"). Rewrote it as the true residual
  case (corpus changed *outside the verb surface* → drift surfaced by a read-only Lint/Survey freshness
  check). Verified C007's sibling phrasings were already aligned (bundle-wide `history` at line 20/37, soft
  out-of-band framing) — no change needed there. Type: update (records + one component edit); no decision
  changed, so no CR/ADR. Linter clean (38 files, 0/0); binary checklist passed.
- **Problem:** ADR005/ADR009 carry `> Superseded in part by…` banners; ADR006
  (caller-orchestrated `reindex`) and ADR008 (per-directory `history` truncation) do not,
  yet ADR011 overturned both. Live C007 contradicts ADR008's still-current-sounding
  "history truncates after relocation."
- **Location:** `drs/ADR006`, `drs/ADR008` (esp. line ~27 truncation consequence); `drs/ADR011`.
- **Fix:** Add the banner convention to ADR006 and ADR008; strike/annotate ADR008's stale
  truncation consequence.
- **Type:** update (records). Coherence fix.

### [x] T2.2 — [Engineering] Survey & Lint have no owning component — will rot
- **Done (2026-07-17, uncommitted):** Design-first via `/codebase-design`, then authored through `/intent`.
  Chose a thin aggregator **component** over a composition-contract section (and over fold-into-skill /
  extend-a-face), because the two cross-face rules are genuine behavior no single face can own and a
  component gives them a test surface + a forcing function. Created **C011 - Curation Operations**
  (`survey`/`lint`, read-only), owning exactly three things: the face→verb **routing table**, the subject
  **rollup rule** (one item per concept; a drift/staleness fact C008 already folded into a retirement
  candidate is not re-emitted raw; routes unioned), and the **conflict-pairing rule** (a `thin`-and-retirement
  concept is one item with `conflict` set and both routes). C011 detects nothing and holds no OKF literal
  (reads only through the faces), so ADR001 stays clean. Mapped C011 to **O004-R003** (the surfacing half),
  reciprocally. Authored `ADR016-survey-lint-aggregation-ownership.md` (component vs contract vs fold vs
  extend-a-face) and `CR008-survey-lint-aggregator-trace.md`; rewrote the README Operations section (the
  "no owning component" sentence is gone), added C011 to the Components list + the O004-R003 map row + See
  Also; named C011 as the aggregator in all seven faces (C004–C010) + added CR008 backlinks; and fixed a
  name collision (ADR015's rejected hypothetical "C011 - Claim Grounding" → unnumbered, since C011 is now
  taken). Linter clean (38 files, 0/0); binary checklist passed. Chosen name "Curation Operations" over the
  finding's "Curation Console" (no interactive surface).
- **Problem:** Aggregation is non-trivial (drift surfaced twice: raw `C006.revalidate_all`
  + folded into `C008.retirement_candidates`; staleness likewise; C009 names a
  "thin *and* retirement-candidate → opposite guidance" conflict owned by nobody). Ingest
  gets a specced C001→C002→C003 pipeline; Survey/Lint get a prose paragraph. Adding a new
  detect-face has no doc forcing the wire-in and no test to catch omission.
- **Location:** `docs/engineering/README.md:29-38`; detect-face notes in C004/C006/C007/C008/C009/C010.
- **Fix:** Add a thin **C011 - Curation Console** (operations aggregator) owning the
  face→verb routing table + cross-face dedup rule; OR at minimum a "Survey/Lint composition"
  contract section with success criteria.
- **Type:** create (component) or update (README). `/codebase-design` candidate.

### [ ] T2.3 — [Engineering] Lint "mutates nothing" but has no read-only freshness interface
- **Problem:** Lint's index-freshness check is specced as "`C005.reindex` idempotence,"
  but `reindex` is C005's only mutation (writes the fix when stale). Read-only Lint must
  either call `reindex` (violates "mutates nothing") or re-derive the index (violates
  one-owner). Undercuts ADR011's "drift eliminated by construction" claim.
- **Location:** `docs/engineering/README.md:31,34-35`; `C005-index-navigation.md:20`; `C007-currency-tracking.md:19`.
- **Fix:** Add `C005.index_status(scope?) → stale_paths[]` (dry-run) and a
  `C007.recency_status` query; have Lint call those.
- **Type:** update.

### [x] T2.4 — [Product] O005 has no retrieval-by-question path
- **Done (2026-07-17, uncommitted):** Added risk **O005-RSK003 - Vocabulary mismatch** (consumer asks
  in terms the knowledge wasn't titled/filed under → present knowledge unreachable by scanning → falls
  back to the one person who holds it, the exact routing the spine forbids) and a solution-free requirement
  **O005-R004 - Question-based retrieval** ("retrieve the concepts relevant to a question posed in their
  own terms — not only browse the listing or follow cross-links"; no mechanism named), plus the map row
  RSK003→R004. O005 discovery now decomposes across browse (R001) / traverse (R002) / integrity (R003) /
  query (R004). Authored `PDR002-question-based-retrieval.md` (four alternatives: fold into the index /
  lean on the agent-skill's inherent reading / scope out for the low-volume envelope / add as its own
  risk+requirement — chosen) and `CR009-question-based-retrieval.md`; added PDR002 + CR009 backlinks to the
  product statement's See Also. Deferred the engineering trace — O005-R004 maps to no component yet, the
  same state O009 held after T1.1 — recorded as a PDR002 consequence and tracked below as **T2.7**. Linter
  clean (40 files, 0/0); binary checklist passed.
- **Problem:** Outcome is "locate knowledge relevant to a *question*," but every
  risk/requirement is browse/traversal (index, cross-links, integrity). The common failure
  — question/vocabulary mismatch to how the corpus is organized — has no risk. This is the
  consumer-facing half of "without routing every question through me."
- **Location:** `docs/product/README.md:97-115`.
- **Fix:** Add O005-RSK003 (vocabulary mismatch / no query path) + a solution-free
  requirement for question-based retrieval.
- **Type:** update.

### [ ] T2.7 — [Engineering · traceability] O005-R004 has no component owner
- **Problem:** T2.4 added requirement O005-R004 (question-based retrieval) with no engineering
  owner — the Requirement-Component Map has no O005-R004 row. Stated but unbuildable-as-traced,
  the same gap O009 had after T1.1 (→ T2.6). A follow-up trace must decide where question-based
  retrieval lives: an extension of C005 - Index & Navigation, a dedicated retrieval component, or
  a capability of the Query operation / curation Agent Skill. Because O005-R004 is solution-free,
  the trace chooses the mechanism (agent reasoning over the corpus vs. a built search/index)
  without reopening the product decision. (Surfaced as a consequence in PDR002; not one of the
  original 14 findings.)
- **Location:** `docs/product/README.md` O005 block; `docs/engineering/README.md`
  Requirement-Component Map; candidate homes C005 and the Query operation.
- **Fix:** Run the engineering trace for O005-R004; add the map row and any component edits.
  `/codebase-design` candidate (where the retrieval seam goes relative to the read-only Query path).
- **Type:** update (map) + likely component edits → CR, possibly an ADR.

### [ ] T2.5 — [Product] O007/O008 share an undeclared dependency on an *optional* charter
- **Problem:** O007 (coverage) leans on "intended scope," but the charter that defines
  scope (O008-R001) is optional. Skip it and O007-R002 coverage review has nothing to
  measure against. (Keep O007/O008 separate — genuine recall vs precision, not redundant.)
- **Location:** `docs/product/README.md` O007 (137-154), O008-R001 (167).
- **Fix:** Add the charter as an explicit dependency on O007-R002's map; state coverage
  review degrades to best-effort when no charter exists.
- **Type:** update.

### [x] T2.6 — [Engineering · traceability] O009 requirements have no component owner
- **Done (2026-07-17, uncommitted):** Ran the engineering trace, design-first via `/codebase-design`.
  Placed all three O009 requirements on **C003 - Integration Authoring** as a self-contained,
  author-time **grounding** capability — `check_grounding(draft) → claim_grounding[]`, shaped exactly
  like C003's `propose_links` (internal, advisory, read-only agent pass). O009-R001 is its non-`supported`
  subset flagged for the steward; O009-R003 is its output presented as the distillation review before the
  write commits; O009-R002 is its verdict persisted as a per-claim **grounding signal**, materialized as a
  terminal verb effect (C004 owns the marker *shape* under OKF's producer-extension allowance §4.2/§9 — no
  OKF change; C003 owns the *value*, the same split C004/C007 have for `timestamp`). Ruled out C002 (no
  authored claim at triage) and C006 (mechanical byte-drift, refuses semantic judgment — a different axis).
  Chose C003-with-named-delegation-seam over a dedicated component (one caller today → hypothetical seam)
  and over an unnamed fold (leaves the judgment untestable/unshareable). Authored
  `ADR015-claim-grounding-seam.md` (alternatives A/B/C) and `CR007-claim-grounding-trace.md`; added the map
  rows O009-R001/R002/R003 → C003; edited C003 (intro, data model, `check_grounding`, behavior, edge cases,
  relationships, success criteria, notes/seam, See Also), C004 (grounding-signal marker convention), the
  engineering README (map, C003 summary, Ingest/Query ops, a fidelity principle, intro sentence, See Also),
  and discharged PDR001's "maps to no component" consequence. Linter clean; binary checklist passed.
- **Problem:** T1.1 added outcome O009 (faithful representation) with three requirements —
  O009-R001 source-grounded authoring, O009-R002 legible grounding, O009-R003 distillation
  review — but none maps to a component in the engineering tree. The outcome is stated but
  unbuildable-as-traced: no owner for the source-grounding check, the consumer-facing
  grounding signal, or the claim-alongside-source review view. (Surfaced as a consequence in
  PDR001; not part of the original 14 findings.)
- **Location:** `docs/product/README.md` O009 block; `docs/engineering/README.md`
  Requirement-Component Map; candidate homes C003 (integration authoring) and C002 (triage).
- **Fix:** Run the engineering trace for O009 — decide whether R001/R003 extend C003's
  authoring path (claim↔source check at author time) and where R002's grounding signal
  lives, then add the rows to the Requirement-Component Map. `/codebase-design` candidate
  (where the fidelity-check seam goes relative to authoring).
- **Type:** update (map) + likely component edits → CR, possibly an ADR.

---

## Tier 3 — Worth doing / polish

### [ ] T3.1 — [Engineering · design win] Five independent corpus-scans, no shared owner
- C002 overlap, C003 `propose_links`, C008 supersession, C009 gap scan, C010 `reconcile`
  are near-identical similarity computations with no shared owner — the scatter the
  one-owner-of-*format* principle avoids. Consider a read-only "corpus analysis" capability
  (or extend C005) they delegate to, keeping each component's *judgment* local. At minimum,
  make the "future delegate to C005" path uniform across the ADRs so it's planned.
- **Type:** design exploration. `/codebase-design` candidate.

### [ ] T3.2 — [Engineering] "Declared snapshot asset" discriminator is unowned
- Relied on by C001/C003/C004/C005 but *what declares it* is owned by nobody; C004 can't
  tell a legit `.md` source-copy under `/references/` from a malformed concept.
- **Fix:** Pin the discriminator in C004 (e.g. "`/references/` is a reserved provenance
  area; files there are assets, never concepts").
- **Location:** `C004-okf-conformance.md:41,53`; `C001:49`; `C003:14`; `C005:14`.

### [ ] T3.3 — [Traceability · systemic] ADR "Affected elements" not backlinked
- ADR006→C003/C008, ADR008→C005, ADR003→C008, ADR004/005/007/009→various: the ADR names
  the element, the element's See Also doesn't reciprocate.
- **Fix:** Pick one convention — "Affected elements" always reciprocates, or demote
  non-contractual mentions to prose — and apply it tree-wide.

### [ ] T3.4 — [Traceability] Assorted smaller link fixes
- CR004 doesn't list the README (whose repair-to-successor principle it introduced).
- O003-R003 map should add C003 (merge remediation) alongside C002.
- ADR005's "union citations" is silently overridden by ADR010 (prune) — banner/annotate.
- C002 still calls the specced C005 "(future)" — `C002-triage.md:60`.

### [ ] T3.5 — [Product] Empty PDR slot; charter not consumer-visible
- Charter-optionality and the coverage/on-scope split are product decisions-with-alternatives
  recorded nowhere (engineering has 13 ADRs; product has 0 PDRs). Author PDRs for at least
  those two.
- Make the charter consumer-visible so an empty result reads "out of scope — here's why"
  rather than sending the consumer back to the steward (extend O008-R001).

---

## Skills that would sharpen the design (via `/ask-matt`)

- **`/codebase-design`** — strongest fit. T1.2 (`merge` seam), T2.2 (Survey/Lint aggregator),
  T3.1 (corpus-scan consolidation) are deep-module questions (where the seam goes, how much
  behavior behind the interface, C003/C008 split leakage). Use before writing component specs.
- **`/domain-modeling`** — for unowned-vocabulary findings: "declared snapshot asset" (T3.2),
  an unnamed shared "corpus analysis" concept (T3.1), the Survey-vs-Lint boundary. Resolves
  overloaded terms; records hard-to-reverse ones as ADRs.
- **`/grilling`** (or `/grill-me`) — optional, to stress-test the PDR-less product decisions
  before writing them (should O009 exist; should the charter be optional; is the 8-outcome
  decomposition right).

---

## Suggested sequencing

1. `/codebase-design` on the `merge` seam (T1.2) and Survey/Lint (T2.2) — design before editing.
2. Tier-1 doc edits: O009 (T1.1), merge contract (T1.2), O004-R003 map/CR003 reconcile (T1.3).
3. Tier-2 coherence + product-coverage edits.
4. Tier-3 polish sweep.

Run the draft → judge loop per batch; commit + CR where the change is material.
