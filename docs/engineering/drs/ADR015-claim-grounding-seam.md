# ADR015 - Claim grounding seam

Place the engineering owner for [PDR001](../../product/drs/PDR001-claim-fidelity-outcome.md)'s fidelity outcome **O009 - Faithful representation** — its three requirements O009-R001 (source-grounded authoring), O009-R002 (legible grounding), and O009-R003 (distillation review) — on **C003 - Integration Authoring** as a self-contained, author-time **grounding** capability, rather than folding it in unnamed or standing up a dedicated fidelity component. The grounding judgment — *does the cited source actually support this claim?* — is modeled as an internal, advisory, read-only capability shaped exactly like C003's existing `propose_links`, with an explicit documented seam for later delegation to a shared grounding owner (or reuse by a post-drift re-check) if a second caller appears.

## Context

[PDR001](../../product/drs/PDR001-claim-fidelity-outcome.md) added **O009 - Faithful representation** as a first-class product outcome: fidelity — whether an authored claim faithfully represents what its cited source supports — is genuinely distinct from traceability (O001) and is the distiller's signature failure mode. PDR001 recorded a standing consequence: *O009's three requirements currently map to no component; a follow-up engineering trace is required to assign owners.* This ADR is that trace.

The three requirements share **one computation** — a heuristic judgment of whether a source supports a claim — and all three anchor at the moment a claim is authored:

- **O009-R001 (source-grounded authoring)** — check each authored claim against its cited source and flag unsupported claims for the steward. The claim must exist and the source must be at hand; both are true only at authoring, where C003 has just composed the claim and promoted the source into `/references/`.
- **O009-R003 (distillation review)** — present each authored claim alongside the source span it was drawn from, *before integration*, so the steward can confirm the distillation. This is a pre-commit recommend-and-confirm surface, the same posture C002 takes with its triage recommendation.
- **O009-R002 (legible grounding)** — surface, with each claim, a signal of how well it is grounded, materialized so any later consumer can read it. Its *value* is the same grounding judgment as R001; it is *written into the concept* (C003 is the only author) and *read* directly via Query.

Two candidate homes are ruled out by the requirements themselves:

- **C002 - Triage** (named as a candidate in the alignment review) adjudicates *raw incoming material* and produces no authored claim — distillation into claims happens downstream in C003. R001/R003 have nothing to check at triage time.
- **C006 - Source Revalidation** compares a source against its baseline for *byte-level drift* and explicitly refuses to judge whether a change matters ("that judgment is the steward's"). Fidelity is a *semantic* judgment about representation, the family of C002's overlap scan and C010's reconciliation (agent passes), not C006's mechanical comparison. C006 checks *has the source moved?*; O009 checks *does the claim match the source?* — different axes.

One constraint governs O009-R002's signal: **Wiki does not define or version the knowledge format** (an OKF change is an OKF change, not a Wiki change). OKF 0.1 resolves this in Wiki's favor — §4.1 permits producer-defined extension keys and §4.2 permits conventional body markers, and §9 requires consumers to tolerate both. So a per-claim grounding signal has a legal home as a **Wiki authoring convention**, which is C004's territory ("the conventions the tool authors to"), with no OKF spec change. This mirrors the lowercase-kebab slug and deprecation-marker conventions C004 already owns.

The only genuinely open design question, then, is narrow: **the grounding judgment has exactly one caller today (C003 at authoring)** — how much future sharing should the placement anticipate?

## Options

- **Own O009-R001/R002/R003 on C003 as a self-contained, named author-time capability, with a documented delegation seam (chosen).** The grounding judgment lives on C003 as an internal, advisory, read-only `check_grounding` — the same shape as C003's `propose_links` and C002's `assess_overlap` (self-contained agent passes with a documented "may later delegate to C005" note, ADR004 precedent). The distillation review (R003) is that assessment presented to the steward with source spans; the grounding signal (R002) is that assessment persisted per claim via a C004-owned marker; the flag (R001) is the non-supported subset surfaced for confirmation. A `## Notes` seam records that a future second caller — a shared corpus-grounding owner (cf. the T3.1 shared-scan concern) or a C006 post-drift re-check — can lift `check_grounding` behind an interface without changing C003's contract. Honors the "one caller = hypothetical seam, don't build it yet" discipline while naming the seam so it does not rot.
- **Stand up a dedicated Claim Grounding component now.** Give the fidelity judgment its own owner immediately, mirroring how O009 got its own outcome: one owner of claim-vs-source support that C003 calls and C006/Survey could reuse. Rejected as premature: there is exactly one caller today, so the seam is hypothetical ("two adapters means a real seam"); it grows the component set for behavior that C003 already has all the inputs for (the composed claim, the promoted source, the citation block); and the self-contained-with-named-seam option captures the same future-sharing path without the standing component. Promote to this if a real second caller lands.
- **Fold R001/R002/R003 into C003's authoring prose with no named capability and no seam.** Simplest edit. Rejected: it leaves the grounding judgment unnamed inside the tree's deepest component, so the fidelity capability has no legible owner to test or later share — the engineering echo of PDR001's own rejected "fold fidelity into O001, where the gap stays hidden inside a passing outcome." When C006 or Survey later needs the judgment, it reappears with no home — the exact scatter the T3.1 finding warns about.

## Decision

Assign **O009-R001, O009-R002, and O009-R003 to C003 - Integration Authoring**, realized as a self-contained author-time **grounding** capability:

- `check_grounding(draft_concept) → claim_grounding[]` — an internal, advisory, read-only agent pass that pairs each authored claim with the cited source span it was drawn from and a support verdict (`supported` / `unsupported` / `partial` / `uncheckable`) with rationale. Self-contained, exactly like `propose_links`: it reads the draft and its promoted source through C004 and writes nothing.
- **O009-R001** is `check_grounding`'s non-`supported` subset, surfaced for the steward as part of authoring's recommend-and-confirm — the fidelity sibling of C003's existing refusal to author a claim *no* source backs (a source present but unsupportive, rather than absent).
- **O009-R003** is `check_grounding`'s output presented as the distillation review — each claim beside its source span and verdict — shown before the write commits, so the steward confirms the distillation.
- **O009-R002** is `check_grounding`'s verdict persisted as a **per-claim grounding signal**, materialized into the concept as a terminal verb effect of `create`/`revise` (alongside recency, history, and reindex) via a **C004-owned marker convention** — a Wiki authoring convention under OKF's producer-extension allowance, not an OKF change. Consumers read it directly via Query.

The grounding check never gates authoring on its own: it flags and surfaces for the steward's judgment (intent in, work out); the steward confirms, edits, or overrides, and only then does the verb write and materialize the signal. Because fidelity checking is heuristic (PDR001), the capability recommends; it does not decide.

A delegation seam is recorded in C003's `## Notes`: `check_grounding` is C003-internal today, and may later be lifted behind a shared grounding interface (or reused by a C006 post-drift re-check) without changing C003's contract — the same forward path `propose_links`/`assess_overlap` hold toward C005.

## Consequences

- O009's three requirements now trace to a component; PDR001's standing follow-up is discharged, and the Requirement-Component Map gains the O009-R001/R002/R003 → C003 rows.
- The fidelity judgment has a named, testable owner (`check_grounding`) with a small interface, without adding a component to the tree — depth on C003, not a new seam maintained for one caller.
- C003 grows one internal capability and one terminal verb effect (the grounding signal); its author-time surface stays a recommend-and-confirm the steward drives. The already-deepest component takes on a second axis of authoring judgment (composition *and* grounding), an accepted cost of keeping fidelity where the claim and source both live.
- C004's convention set gains one marker (the per-claim grounding signal), keeping the one-owner-of-format invariant intact: the signal's *shape* is C004's, its *value* is C003's, exactly as `timestamp` splits between C004 (shape) and C007 (value).
- O009-R002's signal depends on OKF's producer-extension allowance (§4.1/§4.2/§9). A future OKF major version that narrowed extensibility would make this a C004-absorbed change, not a system-wide one — the ADR001 boundary already covers it.
- The delegation seam means a later shared grounding owner (T3.1 corpus-analysis consolidation) or a C006 post-drift re-check is a planned lift of `check_grounding`, not a rediscovery — promote to a dedicated component when a real second caller appears.

## Affected elements

- **C003 - Integration Authoring** — claims O009-R001/R002/R003; gains the `check_grounding` capability, the distillation review, the per-claim grounding signal as a verb effect, and the delegation-seam note. Backlinks here.
- **C004 - OKF Conformance** — owns the per-claim grounding-signal marker as a Wiki authoring convention (shape only, under OKF's extension allowance). Backlinks here.
- **Wiki Curation Architecture** — the Requirement-Component Map gains the O009-R001/R002/R003 → C003 rows; recorded in [CR007](../../crs/CR007-claim-grounding-trace.md).
- **PDR001 - Claim fidelity as a first-class outcome** — its "maps to no component" consequence is discharged by this trace.
