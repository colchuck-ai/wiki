# C008 — Integration

**Role.** The **sole writer** of the concept corpus, via an intent-verb surface: callers
express *intent*, C008 owns every edit — OKF-conformant, cited, grounded, provenance-stamped,
and atomic.

## Capabilities

The intent-verb surface, frozen in [INTERFACES.md § C008](../INTERFACES.md#c008-integration--the-sole-corpus-writer-intent-verb-surface)
(signatures authoritative there; not restated or widened here). Every corpus mutation in the
system is exactly one of these verbs — no other component edits concept files:

- **`integrate(RawMaterial, intent) -> ConceptId`** — distill raw material into a cited,
  grounded, cross-linked concept.
- **`merge(sources, into) -> ConceptId`** — compose overlapping material into one concept,
  pruning redundant citations.
- **`deprecate(ConceptId, reason, successor?)`** — mark a concept deprecated in place, with a
  reason and an optional successor. **This is the supersession path** (see below).
- **`relocate(ConceptId, new_location)`** — move a concept to a new canonical OKF path.
- **`repair_links(ConceptId, successor)`** — retarget inbound links to a successor.
- **`flag(ConceptId, kind, reason)`** — attach a machine-readable scope-divergence /
  retirement-candidate marker.

Callers (C007, C011, C012) are pure decision modules; the *decision* to change the corpus is
theirs, the *edit* is C008's. This is the engineering form of intent-driven authoring
(O004-R001) and assisted upkeep by intent (O004-R003).

**No separate `supersede` verb.** Supersession is `deprecate(id, reason, successor)` — the
`successor` argument records the replacement pointer (O003-R001, O003-RSK002). A supersession
is a deprecation that names what replaces it; splitting it into a second verb would duplicate
the mark-in-place-with-reason behavior for no added expressiveness.

## Data model

C008 owns **the concept corpus** — the single source of truth (DECOMPOSITION.md): plain-text,
OKF-conformant files under version control (O006-R001/R003). It writes these **inline** fields
in each concept file (placement per [ADR004](../drs/ADR004-machine-readable-signal-placement.md)
— a property of the concept a consumer reads lives *in* the concept file, tooling-free):

- **Claims** — the distilled statements the concept asserts.
- **Citations** — a `Citation = { durable_ref, live_locator? }` (shared type) per claim,
  obtained from `C004.retain`; `durable_ref` is the resolution target, `live_locator` the
  distinct drift reference (never a substitute — ADR003).
- **Per-claim grounding signal + review tier** — how well the claim is backed by its cited
  source (from the grounding check, below) and who vetted it (human-reviewed vs. autonomously
  integrated) (O009-R002).
- **Reason-annotated cross-links** — `Link = { target, reason }` (shared type); the stated
  reason is required, not decorative (O005-R002).
- **Deprecation / supersession markers** — a deprecation marker carrying the reason and, for a
  supersession, the successor pointer (O003-R001).
- **Per-concept recency stamp** — the last *meaningful* change time; C008 stamps it because it
  alone knows which mutations are meaningful (O002-R001; consumed by `C005.recency`).

C008 does **not** own the provenance log (that is C005's append-only store) and does **not**
own the derived views (index/graph — C009; drift status — C004→C005). `RawMaterial` and
`ConceptId` are shared types (DECOMPOSITION.md).

## Behavior

**The author-time grounding check — the internal seam behind `integrate`/`merge`, and the
O009 engine.** For each authored claim, before it is admitted:

1. **Resolve the baseline.** Fetch the cited source's durable snapshot via
   `C004.resolve(durable_ref)` — the version-controlled form the claim is grounded *against*,
   never the live origin (ADR003).
2. **Check the claim against the source.** Compare the distilled claim to the resolved content
   and identify the **source span** it was drawn from. The check is **heuristic** — it is a
   proxy for actual faithfulness, which is why O009's true metric is *maximize the proportion*,
   not a guarantee (PDR001). It yields a **grounding signal** (a graded measure) and a
   pass/hold verdict against a threshold.
3. **Decide within the envelope.** Present the claim as a `ProposedAction` to
   `C002.evaluate`. A claim at/above threshold and inside the envelope is **admitted**
   autonomously; one the source does not support, or that falls outside the envelope, is
   **escalated** via `C002.escalate` — with the **source span as `evidence`** so the steward
   sees the claim beside the span it was drawn from (O009-R001, O009-R003). C008 does not
   itself decide act-vs-escalate; that lives only in C002.
4. **Stamp the signal + tier inline.** Write the per-claim grounding signal and the review
   tier: **autonomously integrated** for an envelope-admitted claim, **human-reviewed** for one
   the steward adjudicated through `C002.resolve` (O009-R002). A faithful claim and one that
   outruns its source no longer read identically (O009-RSK001/RSK002).

The seam is internal to C008 (settles old ADR015): callers express intent; grounding is part
of owning the edit.

**Common to every verb.** Each verb (1) normalizes and validates through **C003**
(`apply` then `validate` against the pinned OKF spec — C008 is the only caller of `apply`,
O004-R002/O006-R002); (2) records to **C005** via `record(Disposition)`; and (3) is
**atomic** — the concept-file change(s) and the `C005.record` provenance entry commit
**together, one-commit-or-none** (ADR001). Mutating verbs consult **`C002.evaluate`** wherever
the action is consequential (mechanical edits may act directly; the conservative default is
C002's). C008 never orchestrates derived views — index, graph, and currency **recompute** from
the corpus as the single source of truth.

- **`integrate`** — run the grounding check on each distilled claim; cite via `C004.retain`
  (authoring-time capture, ADR003); author reason-annotated cross-links; conform via C003;
  stamp grounding signal, review tier, and recency; admit/hold/escalate per the seam above.
  Returns the new `ConceptId`. Driven by C007 (`integrate` disposition).
- **`merge`** — compose the overlapping `sources` into the target `into` concept; **prune
  redundant citations** (collapse duplicate `durable_ref`s to one, keeping the surviving claim
  grounded) while **preserving provenance** — the merged-away sources' history is retained
  through the atomic `C005.record` (`outcome = merged`), not discarded. Re-run the grounding
  check on any recomposed claim. Driven by C007 (`merge(into)`).
- **`deprecate`** — mark the concept **deprecated in place** with `reason`; if `successor` is
  given, write a **supersession marker** pointing at it. **Never deletes** — the concept and
  its history remain, now legibly retired (O003-R001, deprecation over deletion). Driven by
  C011.
- **`relocate`** — move the concept to `new_location`, a new canonical OKF path (naming via
  C003). Because inbound links would otherwise dangle, relocate **triggers `repair_links`** so
  they follow — the relocate half of referential integrity (O005-R003, O005-RSK002). Driven by
  C011.
- **`repair_links`** — retarget inbound links to `successor`. Enumerate who points at the
  concept via **`C009.inbound_links`**, then rewrite each inbound concept's cross-link to the
  successor (C008 owns those edits too — it is the sole writer). All touched files plus the
  provenance entry commit in one atomic unit. Driven by C011 (retire/relocate) and by
  `relocate`.
- **`flag`** — attach a machine-readable marker of `kind` (scope-divergence /
  retirement-candidate) with `reason`. Non-destructive metadata only; surfaced to C011/C012 and
  to consumers. Driven by C012 (`scope` divergence) and C011 (retirement candidate).

## Relationships

Matches [INTERFACES.md → Dependency direction](../INTERFACES.md#dependency-direction-no-cycles)
exactly. C008 is the **sole corpus writer**; every arrow *into* it is an **intent** call, never
a shared write.

- **Driven by (intent):** **C007** (`integrate`/`merge`), **C011** (`deprecate`/`relocate`/
  `repair_links`), **C012** (`flag`).
- **Calls (outbound):** **C002** (`evaluate`/`escalate`/`resolve` — governance decisions,
  distillation-review escalations), **C003** (`apply`/`validate` — OKF conformance),
  **C004** (`retain` at authoring, `resolve` for grounding), **C005** (`record` — every
  mutation), **C009** (`inbound_links` — for `repair_links`).
- **Charter (C001):** C008 does **not** read it directly; charter influence reaches C008
  through `C002.evaluate` (which reads C001). C008 executes intent — scope judgment belongs to
  its callers.
- **Derived views recompute; C008 does not orchestrate them** (C009 index/graph, C005
  currency, C004 drift are functions of the corpus — ADR001). No cycle: all C008 edges point
  downward.

## Decisions

- **[ADR001](../drs/ADR001-integration-sole-corpus-writer.md)** — sole corpus writer;
  **per-verb atomicity** (concept change + `C005.record` commit together, one-commit-or-none);
  corpus as single source of truth (C008 does not orchestrate derived views).
- **[ADR003](../drs/ADR003-citation-two-reference-model.md)** — citation captured at
  **authoring time** (`C004.retain` on first cite); grounding reads the `durable_ref`, never
  the live locator.
- **[ADR004](../drs/ADR004-machine-readable-signal-placement.md)** — the inline signals above
  live in the concept file (C008's write); the provenance **log** is C005's, drift is not
  inlined.
- **Grounding check is an internal seam** of C008 (settles old ADR015) — not a separate
  component; owning the edit includes grounding it.

## Success criteria

- **O009 proxy** — *proportion of claims passing the author-time grounding check at or above
  threshold, carried as a per-claim grounding signal.* Every authored claim is checked against
  its resolved source before admission; each carries an inline grounding signal and review
  tier; unsupported claims are escalated with their source span, never admitted silently
  (O009-R001/R002/R003).
- **O001 proxy** — *proportion of claims carrying a citation that resolves to a durable,
  version-controlled form.* Every claim C008 writes carries a `Citation` whose `durable_ref`
  resolves via `C004.resolve` (O001-R001/R002).
- **Atomicity holds:** no corpus mutation exists without its provenance entry, and no
  provenance entry without its mutation — one-commit-or-none, verifiable per commit (ADR001).
- **Sole-writer holds:** every concept-file change in the repository is attributable to a C008
  verb; if any other component is found editing a concept file, that is the failure this
  component exists to prevent.
